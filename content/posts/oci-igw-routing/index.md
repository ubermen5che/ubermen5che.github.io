+++
title = '[Cloud] VCN 네트워크 구조와 패킷 라우팅 메커니즘 분석'
date = 2026-01-26T04:15:00+09:00
draft = false

categories = ["Cloud Engineering", "Oracle Cloud", "Network"]
tags = ["OCI", "VCN", "Troubleshooting", "Packet Flow", "Route Table"]
+++

클라우드 인프라를 코드 레벨(IaC)이나 콘솔에서 수동으로 구축하다 보면, 가장 흔하게 마주하는 문제가 바로 **Connectivity(연결성) 이슈**다. 

최근 Oracle Cloud Infrastructure(OCI)에서 새로운 VCN을 구축하고 인스턴스를 프로비저닝했음에도 불구하고, SSH(`TCP 22`) 접속이 `Operation timed out`으로 실패하는 현상을 겪었다. Public IP가 할당되었고 Security List(방화벽)가 개방되었음에도 발생한 이 문제는, 클라우드 네트워크의 **라우팅 결정(Routing Decision)** 메커니즘을 명확히 이해하지 못했을 때 발생하는 전형적인 구성 오류다.

본 포스트에서는 OCI 네트워크의 핵심 컴포넌트를 아키텍처 관점에서 정의하고, **IGW(Internet Gateway)와 Route Table의 상관관계**를 패킷 흐름 중심으로 분석하여 해결 과정을 정리한다.

## 1. Core Components: OCI 네트워크 아키텍처

문제를 분석하기 전, OCI 네트워크를 구성하는 핵심 컴포넌트의 기술적 정의를 명확히 한다.

### VCN (Virtual Cloud Network)
물리적 온프레미스 네트워크와 논리적으로 격리된 **Software-Defined Network(SDN)**다.
- RFC 1918 사설 IP 대역(CIDR Block)을 기반으로 생성된다.
- Region 단위로 생성되며, 내부적으로 고가용성을 위한 AD(Availability Domain)를 포괄한다.

### Subnet
VCN의 CIDR 블록을 더 작은 단위로 세분화한 **L3 네트워크 세그먼트**다.
- **VNIC 연결의 기준점**이며, 보안 정책(Security List)과 라우팅 정책(Route Table)이 바인딩되는 최소 단위다.
- Public Subnet과 Private Subnet의 구분은 해당 서브넷이 IGW로 향하는 라우팅 경로를 가지고 있느냐에 따라 결정된다.

### VNIC (Virtual Network Interface Card)
인스턴스(VM/Bare Metal)에 부착되는 **가상 네트워크 인터페이스**다.
- 각 VNIC는 서브넷 내의 Private IP를 하나 이상 보유한다.
- 물리적 서버의 NIC와 동일하게 MAC 주소를 가지며, ARP 등 L2 통신을 수행한다.

### IGW (Internet Gateway)
VCN과 Public Internet 간의 트래픽을 중계하는 **소프트웨어 정의 엣지 라우터**다.
- **1:1 NAT 수행:** 인스턴스의 Private IP와 할당된 Public IP 간의 주소 변환(Network Address Translation)을 담당한다.
- 단순한 통로가 아닌, VCN 외부로 나가는 트래픽의 **Next-Hop** 타겟이다.

### Route Table
패킷의 **Destination IP**를 기반으로 다음 경로(Next-Hop)를 결정하는 **Rule Set**이다.
- 각 서브넷은 반드시 하나의 Route Table과 연관(Associate)되어야 한다.
- Default Route(`0.0.0.0/0`) 설정이 없다면, VCN 내부(Local Peering) 외의 트래픽은 라우팅 불가로 드랍(Drop)된다.

---

## 2. Trouble Shooting: SSH 접속 실패 원인 분석

### 시나리오 (Configuration)
- **VCN:** `10.0.0.0/16`
- **Subnet:** `10.0.1.0/24` (Public Subnet으로 의도함)
- **Instance:** `10.0.1.5` (Private) / `150.x.x.x` (Public IP 할당됨)
- **Client:** 로컬 터미널에서 `ssh opc@150.x.x.x` 시도

### 증상 (Symptom)
- SSH 클라이언트에서 `Operation timed out` 발생.
- Security List Inbound `TCP 22` 허용 확인됨.

### 패킷 흐름 분석 (Packet Flow Analysis)
이 문제는 **Return Traffic의 경로 부재**로 인해 발생한다. TCP 3-way Handshake 관점에서 살펴보자.

1.  **Ingress Flow (Client -> Instance):**
    - Client가 `SYN` 패킷을 전송한다.
    - OCI 엣지 라우터가 Public IP(`150.x.x.x`)를 인지하고 IGW로 전달한다.
    - IGW는 NAT를 수행하여 Destination을 Private IP(`10.0.1.5`)로 변환 후 VCN 내부로 인입시킨다.
    - Security List가 허용되어 있으므로 인스턴스의 VNIC까지 패킷이 도달한다.

2.  **Egress Flow (Instance -> Client): <span style="color:red">Failure Point</span>**
    - 인스턴스는 `SYN-ACK` 응답 패킷을 생성한다. Destination은 Client의 Public IP다.
    - 패킷이 서브넷 게이트웨이에 도달하여 **Route Table Lookup**을 수행한다.
    - **문제 발생:** 현재 서브넷에 연결된 Route Table에는 **VCN CIDR(`10.0.0.0/16`)에 대한 Local Rule만 존재**하고, 인터넷 구간(`0.0.0.0/0`)에 대한 규칙이 없다.
    - 라우터는 패킷을 전달할 경로를 찾지 못하고 **Drop (No Route to Host)** 처리한다.

결과적으로 클라이언트는 응답을 받지 못해 Timeout이 발생한다.

### 해결 과정 (Resolution)

문제를 해결하기 위해 명시적인 **Egress Routing Path**를 구성해야 한다.

#### Step 1. Internet Gateway 생성
VCN 레벨에서 IGW 리소스를 생성한다. 이는 논리적인 업링크(Uplink)를 확보하는 과정이다.

#### Step 2. Route Rule 정의
서브넷이 참조할 Route Table에 다음 규칙을 추가한다.
- **Destination:** `0.0.0.0/0` (Default Route)
- **Target Type:** Internet Gateway
- **Target:** `Step 1에서 생성한 IGW ID`

#### Step 3. Route Table Association (핵심)
규칙을 만드는 것만으로는 부족하다. 해당 Route Table을 **Subnet에 명시적으로 매핑(Mapping)**해야 한다.
- OCI Console > Subnet Details > Route Table Configuration 변경.

이 설정이 완료되면, 인스턴스가 보내는 `SYN-ACK` 패킷은 Route Table을 참조하여 IGW로 향하게 되고, IGW는 다시 NAT를 수행하여 인터넷으로 패킷을 방출한다.

---

## 3. Advanced: IGW Route Table Association (Ingress Routing)

IGW 설정 화면의 **"Associate Route Table"** 옵션은 일반적인 웹 서비스 구축 시에는 사용하지 않지만, 보안 아키텍처 설계 시 필수적인 기능이다. 이를 **Ingress Routing**이라 한다.

### 일반적인 라우팅 (Egress Routing)
- **방향:** Subnet -> Internet
- **제어 주체:** Subnet에 연결된 Route Table
- **목적:** 내부 인스턴스가 외부와 통신하기 위함.

### Ingress Routing
- **방향:** Internet -> VCN (Subnet)
- **제어 주체:** **IGW에 연결된 Route Table**
- **목적:** 외부 트래픽이 목적지(Workload)로 도달하기 전, **강제로 보안 어플라이언스(Middlebox)를 경유**하게 만들기 위함.

#### Use Case: Transparent Firewall / IDS
만약 IGW에 Route Table을 연관시키고, `Destination: 10.0.1.0/24 (Subnet) -> Target: 10.0.2.99 (vFirewall VNIC)`와 같은 규칙을 추가한다고 가정하자.

1. 외부에서 `10.0.1.5`로 향하는 트래픽이 들어온다.
2. IGW는 패킷을 바로 서브넷으로 보내지 않고, **연관된 Route Table**을 확인한다.
3. 규칙에 따라 패킷을 `10.0.2.99`(방화벽 인스턴스)로 **Redirect**한다.
4. 방화벽에서 패킷 검사(Inspection) 후 정상이면 실제 목적지로 전달한다.

**주의:** 일반적인 구성에서 이 옵션을 잘못 설정하면, 외부에서 들어오는 트래픽이 의도치 않은 경로로 빠지거나 블랙홀 처리되어 SSH 접속이 차단될 수 있다. 따라서 특별한 보안 요구사항이 없다면 **'None'**으로 두는 것이 정석이다.

---