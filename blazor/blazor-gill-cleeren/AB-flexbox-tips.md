# AB `Flexbox` Tips



## Donner une `max-width` avec `flex-direction: column`

<img src="assets/resolution-width-max-width.png" alt="resolution-width-max-width" style="zoom:150%;" />

Si je crée une pile `flexbox`:

```html
<div class="container">
  <div class="item">
    Lorem ipsum, dolor sit amet consectetur adipisicing elit. Iusto unde dolorem est?
  </div>
  <div class="item">
    Lorem ipsum dolor sit amet consectetur.
  </div>
</div>
```

```css
.container {
  display: flex;
  gap: 12px;
  flex-direction: column;
}

.item {
  padding: 22px;
  background-color: lightblue;
}
```

<img src="assets/basic-flex-column-style.png" alt="basic-flex-column-style" />



En ajoutant `align-items: center`, les éléments prennent la taille de leur contenu:

```css
.container {
  /* ...*/
  align-items: center;
}
```



<img src="assets/flex-column-align-items-center.png" alt="flex-column-align-items-center" />

Je voudrai maintenant pouvoir régler une largeur maximum.

```css
.item {
  /* ... */
  max-width: 460px;
}
```

<img src="assets/stretch-to-max-width.png" alt="stretch-to-max-width" />

Le plus grand élément se rétrécie mais je veux maintenant que le plus petit prenne la taille de `max-width`:

```css
.item {
  /* ... */
  max-width: 460px;
  width: 100%;
}
```

<img src="assets/width-and-max-width-for-better-displkay.png" alt="width-and-max-width-for-better-displkay" />

### Il faut utiliser `max-width` avec `width: 100%`

