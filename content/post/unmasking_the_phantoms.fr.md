+++
title = "Démasquer les fantômes"
date = 2023-09-18T10:00:00+02:00
author = "Mijux"
keywords = ["OSINT", "Stalking", "Blue Team", "Telegram", "Phishing"]
summary = "Utiliser les sites d'hameçonnage pour espionner sur Telegram"
pin = true
draft = false
+++

<style>
    article > * {
        text-align: justify;
    }
</style>

# Démasquer les fantômes : un projet de traque sur Telegram

> **CLAUSE DE NON-RESPONSABILITÉ** : Ce projet est strictement à but éducatif. Il n'approuve ni n'encourage aucune activité illégale ou contraire à l'éthique, y compris l'hameçonnage ou le piratage. Utilisez toujours vos connaissances et vos compétences de manière responsable et dans le respect de la loi. Je n'assume aucune responsabilité en cas d'utilisation abusive ou d'actions illégales découlant des informations présentées ici.

## Résumé 

Ce post traite de l'évolution des méthodologies d'hameçonnage, passant des tactiques conventionnelles basées sur le courrier électronique à des approches plus clandestines utilisant des bots Telegram et des webhooks Discord. Au cœur de ce changement de paradigme se trouve l'acquisition de clés API Telegram, qui permettent de surveiller discrètement les conversations Telegram et d'en extraire des données. La recherche décrit le processus d'obtention de ces clés, explore les subtilités techniques de l'interface avec l'API de Telegram et aborde des considérations pertinentes telles que le mode de protection de la vie privée. Les résultats démontrent l'efficacité du programme dans la surveillance secrète, bien que les limites, en particulier dans la récupération des messages des bots, nécessitent des recherches plus approfondies. Cette étude nous permet de mieux comprendre les techniques contemporaines d'hameçonnage.

## Remerciements

Je tiens à remercier mes amis et ma copine pour les discussions constructives que nous avons eues sur le sujet mais aussi pour les corrections apportées sur cet article.

## Glossaire

- [Telegram](https://telegram.org) : Une application de messagerie mobile et de bureau basée sur le cloud qui met l'accent sur la sécurité et la rapidité.
- [Discord](https://discord.com) : Une application de messagerie mobile et de bureau basée sur le cloud avec des amis et des communautés.
- [OSINT](https://fr.wikipedia.org/wiki/Renseignement_d%27origine_sources_ouvertes) : **O**pen **S**ource **INT**elligence, il s'agit d'extraire des informations publiques de l'Internet.
- [API](https://fr.wikipedia.org/wiki/Interface_de_programmation) : **A**pplication **P**rogramming **I**nterface, l'ensemble des demandes qui peuvent être faites pour interroger un service.
- [clef d'API](https://en.wikipedia.org/wiki/API_key) : Permet d'obtenir les autorisations nécessaires pour interagir avec l'API.
- [webhook](https://fr.wikipedia.org/wiki/Webhook) : Un service qui notifie les utilisateurs. Dans le cas de Discord, il s'agit d'envoyer des messages dans les conversations.
- [bot](https://fr.wikipedia.org/wiki/Bot_informatique) : Abréviation de robot.
- [hameçonnage](https://fr.wikipedia.org/wiki/Hame%C3%A7onnage) : Obtenir des informations personnelles ou sensibles en se faisant passer pour une organisation légitime.

## Introduction

Depuis le COVID-19 et les confinements, nous avons constaté une augmentation des attaques d'hameçonnage. Selon David Warburton sur le [blog F5](https://www.f5.com/company/news/features/phishing-attacks-soar-220--during-covid-19-peak-as-cybercriminal), ces augmentations étaient de l'ordre de 220 % pendant les confinements. En d'autres termes, au lieu d'avoir 100 attaques, nous en avions 320. Aujourd'hui, l'augmentation annuelle est de 15 %. En tant que cyber-curieux, je me suis posé plusieurs questions, dont l'une était la suivante :
- Pourrions-nous traquer les escrocs en utilisant leurs propres sites d'hameçonnage ?

C'est là que tout le projet a vu le jour. Je voulais enquêter et répondre à cette question.

## Objectif

Avant d'expliquer les objectifs du projet, il faut comprendre comment fonctionnent les sites d'hameçonnage. D'une manière générale, ces sites sont simplement des sites PHP qui envoient des courriers électroniques. Cependant, les technologies changent et évoluent. Au lieu d'envoyer des courriels, les pirates utilisent désormais des bots Telegram et des webhooks Discord. Les courriels et les webhooks Discord ne peuvent pas être utilisés aussi efficacement. Bien qu'il soit possible d'effectuer de l'OSINT avec ces courriels, ceux-ci ne sont souvent pas liés à d'autres services. Quant aux webhooks Discord, ils ne permettent pas de récupérer des données, mais seulement d'envoyer des messages.

Il nous reste donc les clés d'API de Telegram. Ces clés sont utilisées pour développer des chatbots sur Telegram, et les chatbots peuvent non seulement envoyer des messages mais aussi **lire** (ce qui est vraiment intéressant pour nous). L'objectif de ce projet est de pouvoir lire les conversations Telegram de manière discrète et d'en extraire des données.

## Logiciel

J'ai été légèrement indulgent en disant, *"nous avons juste besoin des clés API "*, mais la première étape était de trouver un moyen d'obtenir ces insaisissables clés Telegram. En effet, sans l'obtention de ces clés, le projet devient irréalisable.

### Trouver et récupérer les clés de télégramme de l'API

La première étape a été d'obtenir des clés API. Pour être honnête, je n'ai rien créé sur ce point. Un ami m'a parlé d'un projet appelé [StalkPhish](https://github.com/t4d/StalkPhish) qui tente d'extraire le kit d'hameçonnage d'un site web. En quelques mots, ce projet utilise le site [PhishTank](https://phishtank.org/) pour trouver les sites d'hameçonnage encore disponibles. Ensuite, le programme essaie plusieurs points d'accès différents pour trouver les kits d'hameçonnage. Je vous invite à jeter un coup d'œil au projet si vous souhaitez en savoir plus.

<p style="text-align:center;font-style: italic;">C'était finalement la partie la plus facile, je n'ai pas besoin de faire grand-chose de plus.</p>
<div style="display: flex;justify-content: center;">
    <img alt="Sponge Bob All Done" src="https://media.tenor.com/dr760Xwe-xMAAAAd/spongebob-done.gif" width="249" height="184">
</div>


### Comment fonctionne Telegram ?

Après la partie amusante, la partie ennuyeuse. En effet, j'ai dû lire toute la [documentation](https://core.telegram.org/api) pour développeurs de Telegram. Cependant, cette documentation n'était pas suffisante pour faire un code résilient, j'ai dû effectuer de nombreux tests. En particulier, le format JSON des requêtes est facile à utiliser, mais les noms des champs changent en fonction du message. On se retrouve rapidement avec de grosses structures "if" qui ne sont pas très lisibles.

```py
obj = {}

text = msg.get("text")
if text:
    obj["text"] = text

loc = msg.get("location")
if loc:
    obj["location"] = loc

poll = msg.get("poll")
if poll:
    obj["poll"] = poll

contact = msg.get("contact")
if contact:
    obj["contact"] = contact

# etc.
```

Je ne vais pas faire un cours sur "Comment faire un chatbot Telegram", ce n'est pas le but mais certains points sont importants à connaître.

#### L'endpoint getUpdates

Vous pouvez consulter la documentation de l'API [ici](https://core.telegram.org/bots/api#getupdates). Le point intéressant concerne le premier paramètre nommé *offset*. Lorsque je l'ai lu pour la première fois, je me suis dit *"pas de problème, il suffit d'utiliser -1 comme paramètre d'entré"*. Théoriquement, cette solution est viable et je l'ai utilisée dans mon projet. Cependant, prenons une minute pour discuter de cette variable *offset*. Ce nombre augmente de manière séquentielle, donc si le développeur du bot vérifie cette valeur, nous perdons notre discrétion. Cependant, les escrocs utilisent l'API de Telegram uniquement pour envoyer des messages, ils ne l'utilisent pas comme chatbot pour lire les messages.

#### Le mode "Privacy"

Une autre information précieuse de la [documentation](https://core.telegram.org/bots/features#privacy-mode) de Telegram. Nous dépendons beaucoup de la configuration des canaux, en fait si le mode privacy est activé, le bot peut avoir des actions limitées. De plus, nous ne pouvons pas lire les messages du bot. C'est une chose importante à savoir. Nous ne pouvons lire que les messages des utilisateurs du canal, mais nous ne pouvons pas voir les messages des autres bots ni les messages de notre robot. 

Après m'être familiarisé avec l'API, j'ai fini par créer une petite interface web pour faciliter l'utilisation du programme.

## Résultats

Enfin, la partie que vous attendez tous : les résultats !

![Fig.1](/images/posts/utp_fig_1.png)

Comme vous pouvez le voir, mon interface est simple mais efficace. Vous avez sur le panneau de gauche la liste des chatbots et sur le panneau de droite la conversation correspondante. Je gère les messages texte, de localisation, d'image, de vidéo et de voix. Cependant, vous pouvez notifier que j'ai une ligne sans message car je ne gère pas les messages de sondage.

Autre chose super intéressante, chaque message envoyé est enregistré pendant 24 heures sur les serveurs de Telegram, ce qui permet à notre bot de récupérer tous les messages de nos scammers en ne faisant qu'une seule requête par jour.

Pour rappel, nous utilisons le projet StalkPhish pour récupérer le kit d'hameçonnage, puis extraire la clé API de Telegram que nous insérons dans mon programme. Ensuite, vous pouvez espionner n'importe quel canal Telegram pour lequel vous disposez de la clé API du bot. J'ai mis un petit diagramme pour résumer le pipeline.

<div style="display: flex;justify-content: center;">
    <img alt="Fig.2" src="/images/posts/utp_fig_2.png">
</div>


## Limitations

Quelques limitations du projet :
- Impossible de lire les messages du bots.
- Impossible de lire les messages des autres bots.
- Mes conversations se font par bot et non par canal. Les canaux sont donc mélangés les uns avec les autres.
- Nous pouvons être découverts s'ils récupèrent la variable offset obtenu avec l'endpoint getUpdates.
- Nous sommes extrêmement dépendants des clés API de Telegram.

## Conclusion

J'espère que vous avez apprécié ce premier article. Je n'inclurai pas le code source, car vous pouvez déjà consulter le projet StalkPhish, et cela permettra d'éviter que les script kiddies ne fassent n'importe quoi. Cependant, je suis ouvert à vos commentaires et si vous souhaitez en discuter, je serai ravi de le faire. Vous trouverez mon adresse électronique [ici](/fr/).