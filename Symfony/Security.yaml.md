Dans la section encoders, on peut choisir le système de cryptage des mots de passe.

Providers : Normalement, on a un provider par entité user. 

La section 'role_hierarchy' n'est pas créée par défaut. On met les rôles dont on a besoin.

```yaml
role_hierarchy:
# à droite de chaque rôle, on met les rôles inférieurs à celui-ci !
  ROLE_SUPERADMIN: ['ROLE_ADMIN', 'ROLE_MODERATEUR']
  ROLE_ADMIN: ['ROLE_CLIENT']
  ROLE_CLIENT: ['ROLE_USER']
  ROLE_MODERATEUR : ['ROLE_CLIENT']
```

Il existe aussi un ROLE_ALLOWED_TO_SWITCH permettant au SUPERADMIN de changer de rôle. Utile notamment pour le debuggage.

Firewalls: on y touche assez rarement

Access control: On paramètre des accès aux différents rôles en fonction des chemins d'accès.

```yaml
access_control:
#ici, les path sont des expressions régulières. Le chemin ^/admin est ici réservé aux utilisateurs ayant au moins le rôle ADMIN.
  - { path: ^/admin, roles: ROLE_ADMIN }
  - { path: ^/profile, roles: ROLE_USER }
```









