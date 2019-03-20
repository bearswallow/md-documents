`Spring Cloud Gateway` 主要包含两种 Filter -> `GatewayFilter` 和 `GlobalFilter`。

- `GatewayFilter`：作用于当个Route的filter，在每个需要的Route里都需要定义。
- `GlobalFilter`：作用于全部Route的 Filter ，通过

# Global Filter

按照定义的  `org.springframework.core.Ordered` 或者 `@Order` 定义的顺序。

## `LoadBalancerClientFilter` （10100）

