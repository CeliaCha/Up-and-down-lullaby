# Guess the mystery notes

> Lien vers le jeu : [https://simplon-links.firebaseapp.com/](https://simplon-links.firebaseapp.com/)

> Projet réalisé avec Vue.js

Le but du jeu est de deviner la note jouée par l'ordinateur dans la gamme de do majeur. A chaque erreur de l'utilisateur, un indice indique s'il faut chercher une note plus haute ou plus basse. Les notes devinées s'affichent au fur et à mesure sur une portée musicale et le jeu s'arrête au bout de trois erreurs. Le calcul du score inclut un bonus de rapidité.

Principales librairies utilisées : [audiosynth](https://github.com/keithwhor/audiosynth) pour l'interface MIDI, [abcjs](https://github.com/paulrosen/abcjs) pour l'affichage de la portée musicale.

L'essentiel du code est situé dans [/src/App.vue](https://github.com/CeliaCha/Up-and-down-lullaby/blob/master/src/App.vue)

Idées d'améliorations : 
- Ajouter un mode difficile avec les demi-tons
- Autres gammes
- Possibilité d'enregistrer les mélodies obtenues
- Faire deviner les notes d'une mélodie connue
- Améliorer les graphismes (pas mon point fort :-/ )


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
