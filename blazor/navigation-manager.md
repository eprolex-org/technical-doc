# `NavigationManager`

# 📌 **Construction d'`URL`**

| Élément                | Type   | Renvoie               | Exemple                                     |
| ---------------------- | ------ | --------------------- | ------------------------------------------- |
| `Uri`                  | string | URL complète courante | `https://localhost:5001/orders?x=1`         |
| `BaseUri`              | string | Racine de l’app       | `https://localhost:5001/`                   |
| `ToAbsoluteUri()`      | Uri    | URL absolue           | `/products` → `https://…/products`          |
| `ToBaseRelativePath()` | string | Chemin relatif        | `https://…/products/list` → `products/list` |
| `NavigateTo()`         | void   | Change l’URL          | `NavigateTo("/login")`                      |