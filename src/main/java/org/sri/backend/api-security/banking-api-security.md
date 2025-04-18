# Security Mechanisms for a Banking RESTful API

## 1. Authentication
- **OAuth 2.0 + OIDC** (Azure AD, Okta)
- **Short-Lived JWTs** with refresh tokens
- **MFA** for staff/admin

## 2. Authorization
- **RBAC** with Spring Security
- `@PreAuthorize` and method-level checks
- **ABAC** for contextual access (e.g., IP, time)

## 3. Data Protection
- **TLS 1.3** (in transit)
- **AES-256** (at rest)
- **Masking** in responses

## 4. API Security Best Practices
- **Rate Limiting** with Redis
- **Input Validation** using Hibernate Validator
- **Audit Logging** with Spring AOP

## 5. Tooling
- **MuleSoft / Apigee** for API Gateway
- **HashiCorp Vault / GCP Secrets Manager**

## 🧪 Spring Security Sample Code
```java
// Code here
@Configuration
@EnableWebSecurity
public class SecurityConfig {

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .requestMatchers("/api/staff/**").hasAnyRole("STAFF", "ADMIN")
                .requestMatchers("/api/customer/**").hasRole("CUSTOMER")
                .anyRequest().authenticated()
            )
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .decoder(jwtDecoder())
                )
            );
        return http.build();
    }

    @Bean
    public JwtDecoder jwtDecoder() {
        return NimbusJwtDecoder.withJwkSetUri("https://idp.example.com/jwks").build();
    }
}
