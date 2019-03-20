# AccessDecisionManager

```java
/**
 * 授权决策管理器，由它最终做出授权决策
 */
public interface AccessDecisionManager {
    
    /**
     * 通过传递的参数决定是否通过授权
     */
    void decide(Authentication authentication, Object object,
			Collection<ConfigAttribute> configAttributes) throws AccessDeniedException,
			InsufficientAuthenticationException;
    
    /**
     * 判断该授权管理器是否支持指定类型的授权决策
     */
    boolean supports(ConfigAttribute attribute);
    
    /**
     * 判断该授权管理器是否支持指定类型对象的访问授权
     */
    boolean supports(Class<?> clazz);
    
}
```

`AccessDecisionManager` 基本上都不会自己来做具体的授权决策，而是在内部维护一个 `AccessDecisionVoter` 列表，让 `AccessDecisionVoter` 来执行具体的授权投票，`AccessDecisionManager` 再根据投票结果来决定是否通过授权。所以，基本上所有的具体实现都会继承 `AbstractAccessDecisionManager`。

```java
public abstract class AbstractAccessDecisionManager implements AccessDecisionManager,InitializingBean, MessageSourceAware {
    // 授权决策投票者列表
    private List<AccessDecisionVoter<? extends Object>> decisionVoters;
    // 当所有投票者都没有给出确定的投票结果时是否通过授权
    private boolean allowIfAllAbstainDecisions = false;
}
```

目前有三种授权策略

- AffirmativeBased（通过优先）：只要有一个投票者通过则通过授权；否则如果有一个投票者拒绝则拒绝授权。
- ConsensusBased（大多数原则）：需要所有投票者参与投票，大多数投票者（不包含弃权的投票者）的结果就是授权结果；如果通过和拒绝的票数相同且不为0则根据 `allowIfEqualGrantedDeniedDecisions` 设置来判断是否通过授权。
- UnanimousBased（拒绝优先）：只要有一个投票者拒绝则拒绝授权；否则如果有一个投票者通过则通过授权。

==这三种授权策略在所有投票者都弃权的情况下都会根据  `allowIfAllAbstainDecisions` 设置判断是否通过授权。==

# AccessDecisionVoter

```java
/**
 * 访问决策投票者
 */
public interface AccessDecisionVoter<S> {
    // ============= 投票结果只能是下列当中的一个 ==================
    // 通过
    int ACCESS_GRANTED = 1;
    // 弃权
	int ACCESS_ABSTAIN = 0;
    // 拒绝
	int ACCESS_DENIED = -1;
    // =========================================================
}
```

