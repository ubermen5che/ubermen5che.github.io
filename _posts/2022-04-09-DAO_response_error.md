---
layout : posts
comments : true
title: "[Spring] Proxy객체를 Response하려고 할 때 발생하는 오류"

categories:
  - Spring
  - JPA
tags:
  - JPA
  - Proxy Entity
---

## 오류가 발생한 코드

```java
@Transactional
@GetMapping("/group/delete/{id}")
public ResponseEntity<?> groupDelete(@PathVariable Long id, Authentication authentication) {
    Member member = memberService.findByOauthId(authentication.getName()).get();

    List<MemberGroup> memberGroupList = member.getMemberGroupList();

    for (MemberGroup mg : memberGroupList) {
        Group group = mg.getGroup();
        if (group.getId() == id) {
            groupRepository.deleteById(group.getId());
            memberGroupRepository.delete(mg);
            memberGroupList.remove(mg);
            return ResponseEntity.ok().body(group); // 이곳에서 오류 발생
        }
    }

    return ResponseEntity.notFound().build();
}
```

## 오류 메세지

```java
com.fasterxml.jackson.databind.exc.InvalidDefinitionException: No serializer found for class org.hibernate.proxy.pojo.bytebuddy.ByteBuddyInterceptor
and no properties discovered to create BeanSerializer (to avoid exception, disable SerializationFeature.FAIL_ON_EMPTY_BEANS) (through reference chain: MAESIK.demo.domain.Group$HibernateProxy$bGqA2jLK["hibernateLazyInitializer"])
```

## 오류 발생 원인 및 해결방법

jackson 라이브러리로 hybernate proxy객체를 serializing 하려고할 때 오류가 발생.

오류 해결을 위해서 response를 위한 DTO를 따로 정의하고 Builder를 이용하여 DTO에 Entity를 mapping해서 response해주면 해결된다.

```java
@Transactional
    @GetMapping("/group/delete/{id}")
    public ResponseEntity<?> groupDelete(@PathVariable Long id, Authentication authentication) {
        Member member = memberService.findByOauthId(authentication.getName()).get();

        List<MemberGroup> memberGroupList = member.getMemberGroupList();

        for (MemberGroup mg : memberGroupList) {
            Group group = mg.getGroup();
            if (group.getId() == id) {
                groupRepository.deleteById(group.getId());
                memberGroupRepository.delete(mg);
                memberGroupList.remove(mg);
								// 아래는 수정된 코드
                return ResponseEntity.ok().body(GroupResponseDTO.builder()
                        .groupId(group.getId())
                        .groupMasterId(group.getGroupMasterId())
                        .groupExp(group.getGroupExp())
                        .groupName(group.getGroupName())
                        .groupTier(group.getGroupTier())
                        .build());
            }
        }

        return ResponseEntity.notFound().build();
    }
```
