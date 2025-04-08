# 04 Les `roles`

Les `rôles` sont en fait des `claims`.



## Créer un `rôle` en `DB`

```sql
INSERT INTO [dbo].[AspNetRoles]
(
 [Id], [Name], [NormalizedName], [ConcurrencyStamp]
)
VALUES
(
 NEWID(), 'Manager', 'MANAGER', ''
)
```

<img src="assets/new-roles-sql-server-ddb-kkjwszapu.png" alt="new-roles-sql-server-ddb-kkjwszapu" />

On ajoute d'abord un `rôle` manuellement dans la table `AspNetRoles`, puis toujours manuellement on associe un `user` dans la table de jointure `AspNetUserRoles` :

```sql
INSERT INTO [dbo].[AspNetUserRoles]
( 
 	[UserId], 
    [RoleId]
)
VALUES
( 
 	'ae662619-5592-4186-8d21-7d9d6769d485', 
    '03DB352B-3374-457E-81B6-A65F2D70356A'
)
```

<img src="assets/join-table-user-role-added-uuytyyt.png" alt="join-table-user-role-added-uuytyyt" />

On trouve le `UserId` dans la table `AspNetUsers`

<img src="assets/user-taable-get-id-yusffdkjhnb.png" alt="user-taable-get-id-yusffdkjhnb" />



## Ajouter les `rôles` aux `Claims`

`Program.cs`

```ruby
builder.Services.AddIdentityCore<ApplicationUser>(options => options.SignIn.RequireConfirmedAccount = true)
    .AddRoles<IdentityRole>() // <= ici
    .AddEntityFrameworkStores<ApplicationDbContext>()
    .AddSignInManager()
    .AddDefaultTokenProviders();
```

<img src="assets/claims-role-displaying-useer-iiaaghdyueijskc.png" alt="claims-role-displaying-useer-iiaaghdyueijskc" />
