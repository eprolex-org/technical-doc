# 05 Policies

Une `Policy` es un ensemble de règles (`rules`), qui mises ensemble créé une nouvelle `règle`.



## `AddAuthorization`

<img src="assets/policy-built-in-to-us-rrfabbwhd.png" alt="policy-built-in-to-us-rrfabbwhd" />

```ruby
builder.Services.AddAuthorization(
    options => {
		options.AddPolicy(
            "RequireAdminRole", // policy name
            policy => {
            	policy.RequireRole("Admin");
            }
        );
    }
);
```

```ruby
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("AdminAndManagerRoleOnly", policy =>
    {
        policy.RequireRole("Admin")
            .RequireRole("Manager");
    });
});
```



## Dans une `page` : `@attribute [Authorize]`

```ruby
@attribute [Authorize(Policy = "AdminAndManagerRoleOnly")]
```

