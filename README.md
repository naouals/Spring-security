
# Sécurité Spring Security - Gestion Patients

L'implémentation de la sécurité dans l'application de gestion des patients en utilisant Spring Security avec différentes approches d'authentification.

## **Table des Matières**
1. [Description du Projet](#description-du-projet)
2. [Configuration Requise](#configuration-requise)
3. [Strucure du projet](#structure-du-projet)
4. [Configuration clé](#configuration-clé)
5. [Bonnes Pratiques](#bonnes-pratiques)
6. [Captures d'Écran](#captures-décran)

---

## **Description du Projet**
Application web de gestion de dossiers patients avec :
- **Authentification sécurisée** (3 méthodes)
- **Gestion des rôles** (ADMIN/USER)
- **CRUD complet** des patients
- Interface responsive avec Thymeleaf

---

## **Configuration Requise**

### **Prérequis**
- Java JDK 21
- MySQL 8+ H2 (pour le dev)
- Maven 3.6+

### **Dépendances Clés**
```xml
<!-- Spring Security -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>

<!-- Base de données -->
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
            <groupId>com.h2database</groupId>
            <artifactId>h2</artifactId>
            <scope>runtime</scope>
        </dependency>

<!-- Web -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```


---

## **Structure du projet**

```
src/
├── main/
│   ├── java/
│   │   └── org/example/patients/
│   │       ├── controllers/       # Contrôleurs
│   │       ├── entities/          # Entités JPA
│   │       ├── repositories/      # Repositories
│   │       └── security/          # Configuration sécurité
│   ├── resources/
│   │   ├── templates/             # Vues Thymeleaf
│   │   ├── static/                # Assets
│   │   └── application.properties # Config
└── schema.sql
```

---

## **Configuration Clé**

**SecurityConfig.java** (Cœur de la sécurité) :
```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity(prePostEnabled = true)
public class SecurityConfig {
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http
                .formLogin(form -> form
                        .loginPage("/login").permitAll())
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/admin/**").hasRole("ADMIN")
                        .requestMatchers("/user/**").hasRole("USER"))
                .exceptionHandling(e -> e
                        .accessDeniedPage("/notAuthorized"))
                .build();
    }
}
```
**Rôle** :  
- Active la sécurité web et les annotations `@PreAuthorize`  
- Configure les règles d'accès par rôle  
- Redirige vers une page custom en cas d'erreur 403  

![image](https://github.com/user-attachments/assets/d901af4d-b0d5-4d63-97ed-42263503b636)

---

### Méthodes d'Authentification

### a) InMemory (Développement)
```java
@Bean
public InMemoryUserDetailsManager inMemoryUsers() {
    return new InMemoryUserDetailsManager(
        User.withUsername("admin")
            .password(passwordEncoder.encode("1234"))
            .roles("ADMIN", "USER").build()
    );
}
```

### b) JDBC (Production)
```sql
-- schema.sql
CREATE TABLE users (
    username VARCHAR(50) PRIMARY KEY,
    password VARCHAR(500) NOT NULL,
    enabled BOOLEAN NOT NULL
);
CREATE TABLE authorities (
    username VARCHAR(50) REFERENCES users(username),
    authority VARCHAR(50) NOT NULL
);
```

### c) UserDetailsService (Personnalisé)
```java
@Service
public class UserDetailServiceImpl implements UserDetailsService {
    @Override
    public UserDetails loadUserByUsername(String username) {
        AppUser user = accountService.loadUserByUsername(username);
        return new User(
            user.getUsername(),
            user.getPassword(),
            user.getRoles().stream()
                .map(r -> new SimpleGrantedAuthority(r.getRole()))
                .toList()
        );
    }
}
```

---

### Sécurité dans les Vues

**navbar.html** (Contrôle d'accès) :
```html
<div sec:authorize="hasRole('ADMIN')">
    <a th:href="@{/admin/formPatient}">Nouveau Patient</a>
</div>
```
```html
<td th:if="${#authorization.expression('hasRole(''ADMIN'')')}">
        <a class="btn btn-danger btn-sm"
           th:href="@{/admin/delete(id=${p.id},page=${currentPage},keyword=${keyword})}"
           onclick="return confirm('Confirmer la suppression ?')"><i class="bi bi-trash"></i> Supprimer</a>
        <a class="btn btn-warning btn-sm"
           th:href="@{/admin/editPatient(id=${p.id})}"><i class="bi bi-pencil"></i> Modifier</a>
      </td>
```
Les fonctionnalités 'modifier', 'supprimer' et 'ajouter un patient' sont autorisées uniquement pour l'admin.

![image](https://github.com/user-attachments/assets/276c5665-9ebf-4795-85b5-eda22431a8fe) 
![image](https://github.com/user-attachments/assets/7f9c016b-2009-4ede-8e53-bfadd596b2d1)


**login.html** (Formulaire custom) :
```html
<form th:action="@{/login}" method="post">
    <input type="text" name="username" placeholder="Nom d'utilisateur">
    <input type="password" name="password" placeholder="Mot de passe">
    <button type="submit">Connexion</button>
    <div th:if="${param.error}" class="alert">Identifiants invalides</div>
</form>
```

![image](https://github.com/user-attachments/assets/cd9a6b06-a861-4c28-9b55-396d42c9b999)

---



## Bonnes Pratiques

1. **Séparation des rôles** :  
   - `/user/**` pour les utilisateurs standards  
   - `/admin/**` pour les administrateurs

2. **Protection CSRF** :  
   Activée par défaut dans Spring Security

3. **Hachage des mots de passe** :  
   Utilisation de BCryptPasswordEncoder

4. **Messages sécurisés** :  
   Messages d'erreur génériques pour éviter les fuites d'information

---

## **Captures d'Écran**

### **1. Flux d'Authentification**
![image](https://github.com/user-attachments/assets/46230d48-2e19-4db1-937d-924624e93a03)

*Page de connexion personnalisée avec gestion des erreurs*



### **2. Console BDD** :
   ![image](https://github.com/user-attachments/assets/0f2ff706-2ad9-45b5-b2d2-369ddb345288)

   *la base de données 'patients_db'*

![image](https://github.com/user-attachments/assets/bc0bbfad-8203-46aa-90bc-0000a6f353bd)
![image](https://github.com/user-attachments/assets/af22bc7c-0d17-46b4-8ef9-fbb440b9ffe9)
![image](https://github.com/user-attachments/assets/64f0ccb6-52d0-47cb-a599-117a18dd8fda)


   *- Tables `users` et `authorities` pour JDBC*

   ![image](https://github.com/user-attachments/assets/858e8732-5076-4b51-94f6-573a63dc12a6)
   ![image](https://github.com/user-attachments/assets/202e49be-beb5-4dfb-a4e9-70e7c4fa8ead)
   ![image](https://github.com/user-attachments/assets/6d18bd9f-d34a-49b1-b8aa-ab83ed7e8f44)
   ![image](https://github.com/user-attachments/assets/e36543ed-a8b6-48ec-b4ed-5f5225693d6c)


   - Tables `app_user`, `app_role`  et `app_suer_roles` pour l'approche personnalisée

---
