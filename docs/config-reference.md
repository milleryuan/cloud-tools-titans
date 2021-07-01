## TitanSideCars (root)

```yaml
# titanSideCars.

  imageRegistry:  string

  envoy:          Envoy
  opa:            OPA
  ratelimit:      Ratelimit
  ingress:        Ingress
  egress:         Egress
```

### imageRegistry
(string, required) Common docker image registry path.

### envoy
([Envoy](), required) Section to enable and configure Envoy sidecar	

### opa
([OPA](), optional) Section to enable and configure OPA sidecar

### ratelimit
([Ratelimit](), optional) Section to enable and configure global ratelimiting sidecar	

### ingress
([Ingress](), optional) Section to configure processing http inbound requests

### Egress
([Egress](), optional) Section to configure processing http outbound requests


---

## Envoy
High level envoy settings

```yaml
# titanSideCars.envoy.

  enabled:        bool
  imageRegistry:  string
```
### enabled
(bool, default true) Set to false to disable envoy sidecar

### imageRegistry
(string, optional) Override docker image registry path used for envoy sidecar	

---


## OPA
High level settings related to opa sidecar

```yaml
# titanSideCars.opa.

  enabled:        bool
  imageRegistry:  string
```

### enabled
(bool, default true) Set to false to disable opa sidecar

### imageRegistry
(string, optional) Override docker image registry path used for opa sidecar	

---

## Ratelimit
High level settings related to ratelimit sidecar

```yaml
# titanSideCars.ratelimit.

  enabled:        bool
  imageRegistry:  string
```

### enabled
(bool, default true) Set to false to disable ratelimit sidecar

### imageRegistry
(string, optional) Override docker image registry path used for ratelimit sidecar	

---

## Ingress

```yaml
# titanSideCars.ingress.
  tokenCheck:     bool
  accessPolicy:   IngressAccessPolicy
  routes:         []IngressRoute
```

### tokenCheck
(bool, default false) Controls token validation. If set to true, token validation is performed on all incoming requests. Token validation can be skipped on a per route basis. If set to false, token validation is skipped by default unless enabled for specific requests on a per route basis.

### accessPolicy
([IngressAccessPolicy](), optional) Configures the default access check behaviour for all incoming requests

### routes
([][IngressRoute](), optional)


## IngressRoute

```yaml
# titanSideCars.ingress.routes[]

  # route definition
  match:            RouteMatch

  # per route actions
  tokenCheck:       bool
  metrics:          PerRouteMetrics
  accessPolicy:     PerRouteAccessPolicy
  ratelimit:        PerRouteRatelimit
  route:            RoutingAction
```

### match
([RouteMatch](), required) Route matching parameters. 

### tokenCheck
(bool, optional) Controls enablement of token validation for matching requests. This flag works in conjunction with `titanSideCars.ingress.tokenCheck` flag. <br />
If ingress level flag is set to true then setting this to false with disable token validation for matching requests. <br />
If ingress level flag is set to false then setting this to true with enable token validation for matching requests.

### metrics
([RouteMetrics](), optional) If supplied, triggers metrics for matching requests

### accessPolicy
([RouteAccessPolicy](), optional) If supplied, triggers access policy enforcement for matching requests.

### ratelimit
([RouteRatelimit](), optional) If supplied, triggers ratelimit policy enforcement for matching requests.

### route
([RoutingAction](), optional) Specifies routing destination for matching requests. If un-specified, matching requests are routed to  `local-myapp`

---

## RoutingAction

```yaml
# titanSideCars.ingress.routes[].route.
# titanSideCars.egress.routes[].route.

  cluster: string
  prefixRewrite: string
```

### cluster
(string, required) Cluster name to route the request to

### prefixRewrite
(string, optional) 



**TODO - INCOMPLETE**

---

## PerRouteMetrics

```yaml
# titanSideCars.ingress.routes[].metrics.
# titanSideCars.egress.routes[].metrics.

  enabled:  bool
  name:     string
```
### enabled
(bool, default true) Controls generation of metrics for corresponding route definition

### name
(string, required) Metric name. Metrics are output in `vhost.<service-name>-ingress.vcluster.<metric-name>.` and `vhost.<service-name>-egress.vcluster.<metrics-name>.` namespaces for requests in `ingress` and `egress` paths respectively. 

---

## PerRouteRatelimit

```yaml
# titanSideCars.ingress.routes[].ratelimit.

  enabled:  bool
  actions:  []RatelimitAction
```

### enabled
(bool, default true) Controls enforcement of supplied ratelimit actions

### actions
([]RatelimitAction, required) One or more ratelimit actions on request that match corresponding route definiton. Request is matched against all actions and each matching action is evaluated independently.  

---

### RatelimitAction

```yaml
# titanSideCars.ingress.routes[].ratelimit.actions[].

  descriptors: []RatelimitDescriptor
  limit: string
```

### descriptors
([][RatelimitDescriptor](), optional) List of descriptors that define the attributes to ratelimit on. Corresponding route definition is an implicit descriptor. Incoming request must match all descriptors to be considered for ratelimiting. The ordering of descriptors is not relevant.

### limit
(string, required) Ratelimit value in `<ratelimit-per-unit>/<unit>` format. Supported units are `second` `minute` `hour` and `day`.

Example: *5/second*, *50/minute*, *100/hour*, *10/day*

Instead of supplying a hard-coded limit, limit can also refer to a key from `titanSideCars.ratelimit.limits` key-value pairs as in example below. This pattern allows for easy configuration of limits on a per environment basis.

<details>
  <summary>Click to expand!</summary>
  
```yaml
titanSideCars:
  ratelimit:
    limits:
      small: 100/day
  ingress:
    routes:
    - match: /
      ratelimit:
        actions:
        - limit: small
```
</details>

---

### RatelimitDescriptor

```yaml
# titanSideCars.ingress.routes[].ratelimit.actions[].descriptors[].

  key:    string

  # comparison operators
  eq:     string          # equals
  sw:     string          # starts-with
  ew:     string          # ends-with 
  co:     string          # contains
  lk:     string          # like
  pr:     bool            # present
  neq:    string          # not equals
  nsw:    string          # not starts-with
  new:    string          # not ends-with
  nco:    string          # not contains
  nlk:    string          # not like
  npr:    bool            # not present
```

#### key
(string, required) Attribute to rate-limit on. If no comparison operator is specified, ratelimit is performed on each unique value of the attribute. Key may refer to a header, token claim or an attribute from json payload.

A header may be referenced via `header.` notation. Example: `header.x-request-id` <br />
A token claim may be referenced via `token.` notation. Example: `token.sub.scope` <br />
A payload attribute may be referenced via `payload.` notation. Example: `payload.userType` <br />

#### eq | sw | ew | co | lk | pr | neq | nsw | new | nco | nlk | npr
(oneof optional) Comparison operator. When specified, the ratelimit is performed on result of comparision. <br />
For binary operators, the operator value indicates the right hand operand and should be a string. 

The supported operators are
- **eq/neq**: Exact string match
- **sw/nsw**: String prefix match
- **ew/new**: String suffix match
- **co/nco**: Substring match
- **lk/nlk**: Regex match. Regex syntax is documented [here](https://github.com/google/re2/wiki/Syntax)
- **pr/npr**: Unary operator. Indicates if key is present or not. Only `true` value is used. Use `pr` to test presence and `npr` to test non presence.

---

## IngressAccessPolicy

```yaml
# titanSideCars.ingress.accessPolicy.

  defaultAction:  enum
```

### defaultAction
(enum, default ALLOW) Valid values are `ALLOW` or `DENY`. 

If defaultAction is set to `ALLOW` then request is allowed by default. The request is denied if it matches one or more access policies defined on a per route basis. <br />
If defaultAction is set to `DENY` then request is denied by default. The request is allowed if it matches one or more access policies defined on a per route basis. <br />
An http status code `403 (Forbidden)` is returned on denial.


**TODO**: Add info regarding processing order 

---

## PerRouteAccessPolicy

```yaml
# titanSideCars.ingress.routes[].accessPolicy.

  enabled:  bool
  name:     string
  oneOf:    []AccessRuleSet  
```

### enabled
(bool, default true) Controls enforecement of supplied access policy

### name
(string, optional) Policy name to enhance readability of generated policy. If not supplied, a unique policy identifier gets auto-generated

### oneOf
([]AccessRuleSet, optional) A list of rulesets that collectively define the access policy for requests that match corresponding route definiton. The route definition is implicitly part of each ruleset. If unspecified, the policy is generated from route defintion alone.

A request a said to `match` the policy if it matches atleast `one-of` the rulesets

---

## AccessRuleSet

```yaml
# titanSideCars.ingress.routes[].accessPolicy.oneOf[].

  allOf: []AccessRule
```

#### allOf
([]AccessRule, required) A set of rules that collectively define an access ruleset.

A request a said to `match` the ruleset if it matches `all-of` the rules in the ruleset

---

## AcessRule

```yaml
# titanSideCars.ingress.routes[].accessPolicy.oneOf[].allOf[].

  key:    string              # left operand

  # operator + right operand
  eq:     string              # equals
  sw:     string              # starts-with
  ew:     string              # ends-with 
  co:     string              # contains
  lk:     string              # like
  pr:     bool                # present (unary)
  neq:    string              # not equals
  nsw:    string              # not starts-with
  new:    string              # not ends-with
  nco:    string              # not contains
  nlk:    string              # not like
  npr:    bool                # not present (unary)
```
An access rule is a simple expression of the form `'left-operand operator right-operand'` and follows standard rules of expression evaluation. The expression is capable of comparing headers, claim from token, json payload attributes, and raw text values.

A header can be referenced via `header.` notaion. Example: *header.x-request-id* <br />
A token claim can be referenced via `token.` notation. Example: *token.sub.scope* or *token.jti* <br />
A payload attribute can be referenced via `payload.` notation. Example: *payload.userType* <br />

If operand has none of the special prefixes, it is treated as raw text

### key
(string, required) Left hand operand. The left operand can be a header, a token claim, or a json payload attribute


### eq | sw | ew | co | lk | pr | neq | nsw | new | nco | nlk | npr
(oneof required oneof) Comparison operator. For binary operators, the operator value indicates the right hand operand.

The supported operators are
- **eq/neq**: Exact string match
- **sw/nsw**: String prefix match
- **ew/new**: String suffix match
- **co/nco**: Substring match
- **lk/nlk**: Regex match. Regex syntax is documented [here](https://github.com/google/re2/wiki/Syntax)
- **pr/npr**: Unary operator. Indicates if key is present or not. Only `true` value is used. Use `pr` to test presence and `npr` to test non presence.

The right hand operand can be a header, a token claim, a json payload attribute, or raw text

---


## RouteMatch

```yaml
  prefix:       string
  regex:        string
  method:       string
  notMethod:    string
  headers:      []HeaderMatch
```

### prefix | regex
(string, optional oneof) Specifies type of match to be performed on url path
- **prefix**: Prefix matching `:path` header. The `:path` header contains entire url path including the query params.
- **regex**: Regex matching entire `:path` header. The regex must constructed to include query parameters. Regex string must adhere to documented [syntax](https://github.com/google/re2/wiki/Syntax)

If neither `prefix` nor `regex` is supplied then match is performed on `/` prefix

### method | notMethod
(string, optional oneof)
- **method**: Request must match supplied value like GET, POST etc. 
- **notMethod**: Request must not match supplied value


### headers
([][HeaderMatch](), optional) List of http headers to match

---

## HeaderMatch

```yaml
  key:    string

  eq:     string              # equals
  sw:     string              # starts-with
  ew:     string              # ends-with 
  co:     string              # contains
  lk:     string              # like
  pr:     bool                # present (unary)
  neq:    string              # not equals
  nsw:    string              # not starts-with
  new:    string              # not ends-with
  nco:    string              # not contains
  nlk:    string              # not like
  npr:    bool                # not present (unary)
```

### key
(string, required) Header name

### eq | sw | ew | co | lk | pr | neq | nsw | new | nco | nlk | npr
(oneof required oneof) Comparison operator. For binary operators, the operator value indicates the right hand operand and should be a raw string. 

The supported operators are
- **eq/neq**: Exact string match
- **sw/nsw**: String prefix match
- **ew/new**: String suffix match
- **co/nco**: Substring match
- **lk/nlk**: Regex match. Regex syntax is documented [here](https://github.com/google/re2/wiki/Syntax)
- **pr/npr**: Unary operator. Indicates if key is present or not. Only `true` value is used. Use `pr` to test presence and `npr` to test non presence.

---