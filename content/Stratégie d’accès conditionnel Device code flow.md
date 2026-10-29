## Qu’est-ce que le device code flow ?

Voici un schéma issu de la documentation Microsoft qui résume parfaitement le mécanisme “Device flow code”.

![[Pasted image 20261029114143.png]]

> [!info]Device flow code est en réalité un mécanisme permettant de se connecter à des appareils via un lien URL afin de s’authentifier. Une fois l’utilisateur connecté, l’appareil peut obtenir des jetons d’accès et les actualiser au besoin.

Selon le blog de microsoft [https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/#Update-February-14](https://www.microsoft.com/en-us/security/blog/2025/02/13/storm-2372-conducts-device-code-phishing-campaign/#Update-February-14)

Cette attaque, non nouvelle (première apparition en août 2024), est utilisée dans le but de dérober le code de connexion des appareils et de récupérer le jeton valide à l'intérieur d'une session valide ; l'attaquant peut donc aisément se déplacer latéralement. Cette attaque se nomme "Storm-2372" selon le blog de Microsoft.

Voici le schéma de cette attaque :

![[Pasted image 20261029114207.png]]

Afin de vous prémunir contre ce type d’attaque (AitM), Microsoft recommande certaines actions :

- [ ] Bloquer le device code flow (dans la mesure du possible). Dans certains cas, il est nécessaire, par exemple pour Microsoft Teams Room. Mais dans le cas contraire, il est recommandé de restreindre au maximum le device code flow.
- [ ] Révoquer les actualisations de jeton.
- [ ] Définir un accès conditionnel pour forcer la réauthentification des utilisateurs.
- [ ] Définir une politique de risque de connexion.

Simuler une attaque de token est très simple. Voici le flux :

> [!info]En utilisant le module TokenTactitcs, vous allez pouvoir via la commande “Get-AzureToken -Client MSGraph” récupérer le code pour vous authentifier sur l’adresse suivante [Connectez-vous à votre compte](https://login.microsoftonline.com/common/oauth2/deviceauth)

Exemple :

![[Pasted image 20261029114219.png]]

![[Pasted image 20261029114230.png]]

## ETAPE 1 : Blocage device code flow via accès conditionnel.

Se rendre sur [https://entra.microsoft.com/](https://entra.microsoft.com/) → accès conditionnel.

![[Pasted image 20261029114238.png]]

Inclure les utilisateurs et surtout, pensez à exclure les utilisateurs concernés.

![[Pasted image 20261029114251.png]]

Ressources cibles : inclure et exclure les applications.

Conditions : cocher la case Flux de code d’appareil (device code flow).

![[Pasted image 20261029114258.png]]

Dans un premier temps, je vous conseille d’activer votre stratégie en mode “rapport uniquement”, pour ne pas bloquer des accès non évalués avant cette mise en place.

Résultat :

![[Pasted image 20261029114308.png]]

Vérification dans les rapports de connexion :

  ![[Pasted image 20261029114319.png]]

![[Pasted image 20261029114326.png]]



## ÉTAPE 2 : Exiger une réauthentification de session via les accès conditionnels.

Protégez l’accès utilisateur sur les appareils non gérés en empêchant les sessions de navigateur de rester connectées une fois le navigateur fermé et en définissant une fréquence de connexion sur 1 heure.

Se rendre sur [https://entra.microsoft.com/](https://entra.microsoft.com/)

![[Pasted image 20261029114336.png]]

Inclure / exclure vos groupes.

Ressources / inclure / exclure les applications que vous souhaitez.

Conditions : Définir une règle de syntaxe ici device.trustType -ne "ServerAD" -or device.isCompliant -ne True

Vous pouvez activer ou juste en mode rapport dans un premier temps.

## ETAPE 3 : Session de navigateur permanente

Se rendre sur [https://entra.microsoft.com/](https://entra.microsoft.com/)

![[Pasted image 20261029114346.png]]

Pour cette stratégie, Microsoft recommande le type de ressources “toutes les ressources (anciennement toutes les applications cloud)”.

![[Pasted image 20261029114401.png]]

![[Pasted image 20261029114409.png]]

Vous pouvez définir des conditions spécifiques ici : risque de connexion / risque utilisateur évoqué dans cet article [https://www.linkedin.com/pulse/zero-trust-hardening-entra-id-workload-standarduser-billy-stucke-jd1ze/?trackingId=xIItZOTBQN%2BSYWFs0ta8mw%3D%3D](https://www.linkedin.com/pulse/zero-trust-hardening-entra-id-workload-standarduser-billy-stucke-jd1ze/?trackingId=xIItZOTBQN%2BSYWFs0ta8mw%3D%3D)

> [!warning]Je conseille de ne pas mélanger ces stratégies et d’en voir une par besoin.

![[Pasted image 20261029114420.png]]

En conclusion, cet article présente une analyse approfondie de la menace "Storm-2372" liée au device flow code et propose une stratégie de protection en trois étapes:

1. La mise en place d'un blocage du device flow code via l'accès conditionnel, avec une recommandation de commencer en mode "rapport uniquement".
2. L'implémentation d'une réauthentification de session obligatoire, particulièrement importante pour les appareils non gérés.
3. La gestion des sessions de navigateur permanentes avec des contrôles spécifiques.

Cette approche en plusieurs couches permet de renforcer significativement la sécurité contre les attaques de type device code flow, tout en maintenant un équilibre entre sécurité et utilisabilité grâce à une mise en place progressive et contrôlée des mesures.