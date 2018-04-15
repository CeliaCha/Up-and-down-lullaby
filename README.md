# UpAndDown-Lullaby

> Lien vers le jeu : [https://simplon-links.firebaseapp.com/](https://simplon-links.firebaseapp.com/)

> A Vue.js project

Le but du jeu est de deviner la note jouée par l'ordinateur. A chaque erreur de l'utilisateur, un indice indique s'il faut chercher une note plus haute ou plus basse. Les notes devinées s'affichent au fur et à mesure sur une portée musicale et le jeu s'arrête au bout de trois erreurs.

Librairies utilisées : [audiosynth](https://github.com/keithwhor/audiosynth) pour l'interface MIDI, [abcjs](https://github.com/paulrosen/abcjs) pour l'affichage de la portée musicale.

L'essentiel du code est situé dans /src/App.vue


## Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:8080
npm run dev

# build for production with minification
npm run build

# build for production and view the bundle analyzer report
npm run build --report
```
