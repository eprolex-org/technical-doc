# 03 `HubContext`

À la place d'appeler directement mon `Hub`, je peux placer la logique dans un `endpoint` et renvoyer grâce au `HubContext` un événement aux différents clients `SignalR`.



## Dans le `Server`

### Création d'un `Hub` vide

