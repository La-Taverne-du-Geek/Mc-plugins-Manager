# Gestionnaire de Plugins Minecraft pour Pterodactyl

## 🧭 Bienvenue

Cette extension simplifie l’installation et la gestion des plugins **Minecraft** directement depuis votre **Panel Pterodactyl**.

---

## ⚙️ Installation

Avant de commencer, assurez-vous que votre **Panel Pterodactyl** est en version **v1.11.x**.

### 1️⃣ Téléversement des Fichiers

Dans le dossier de l’extension, ouvrez le dossier `pterodactyl`, où vous trouverez trois répertoires :  
`app`, `database` et `resources`.  
Téléversez ces répertoires à la racine de votre installation Pterodactyl (généralement dans `/var/www/pterodactyl`).

---

### 2️⃣ Modification des Routes Frontend

**Ouvrez le fichier :** `resources/scripts/routers/routes.ts`

**Étape 1 :** Trouvez la ligne suivante :

```js
import ServerActivityLogContainer from "@/components/server/ServerActivityLogContainer";
```

**Ajoutez en dessous :**

```js
import PluginsManagerContainer from "@/components/server/mcplugins/PluginsManagerContainer";
```

**Étape 2 :** Recherchez ce bloc :

```js
{
    path: '/files',
    permission: 'file.*',
    name: 'Files',
    component: FileManagerContainer,
},
```

**Ajoutez ce bloc juste après :**

```js
{
    path: '/mcplugins',
    permission: 'file.*',
    name: 'Plugins',
    component: PluginsManagerContainer,
},
```

---

### 3️⃣ Modification des Routes de l’API Backend

**Ouvrez le fichier :** `routes/api-client.php`

**Trouvez ces lignes :**

```php
Route::post('/command', [Client\Servers\CommandController::class, 'index']);
Route::post('/power', [Client\Servers\PowerController::class, 'index']);
```

**Ajoutez ce bloc juste après :**

```php
Route::group(['prefix' => '/mcplugins'], function () {
    Route::get('/', [Client\Servers\MCPlugins\PluginsManagerController::class, 'index']);
    Route::get('/version', [Client\Servers\MCPlugins\PluginVersionsController::class, 'index']);
    Route::post('/install', [Client\Servers\MCPlugins\InstallPluginsController::class, 'index']);
    Route::get('/settings', [Client\Servers\MCPlugins\MCPluginsSettingsController::class, 'index']);
});
```

---

### 4️⃣ Modification des Routes d’Administration

**Ouvrez le fichier :** `routes/admin.php`

**Ajoutez ce bloc à la fin du fichier :**

```php
Route::group(['prefix' => 'mcplugins'], function () {
    Route::get('/', [Admin\MCPlugins\MCPluginsController::class, 'index'])->name('admin.mcplugins');
    Route::post('/', [Admin\MCPlugins\MCPluginsController::class, 'update'])->name('admin.mcplugins.update');
});
```

---

### 5️⃣ Modification du Menu d’Administration

**Ouvrez :** `resources/views/layouts/admin.blade.php`

**Trouvez ce bloc :**

```php
<li class="{{ ! starts_with(Route::currentRouteName(), 'admin.nests') ?: 'active' }}">
    <a href="{{ route('admin.nests') }}">
        <i class="fa fa-th-large"></i> <span>Nests</span>
    </a>
</li>
```

**Ajoutez ce bloc juste après :**

```php
<li class="{{ ! starts_with(Route::currentRouteName(), 'admin.mcplugins') ?: 'active' }}">
    <a href="{{ route('admin.mcplugins') }}">
        <i class="fa fa-cubes"></i> <span>MC Plugins</span>
    </a>
</li>
```

---

### 6️⃣ Commandes à Exécuter

Dans votre répertoire principal de Pterodactyl (`/var/www/pterodactyl`), exécutez les commandes suivantes :

```bash
php artisan route:clear
php artisan cache:clear
php artisan migrate --seed --force
chmod -R 777 /var/www/pterodactyl
```

---

### 7️⃣ Configuration de l’API CurseForge

Le gestionnaire de plugins Minecraft utilise l’API **CurseForge** pour accéder à une vaste bibliothèque de plugins.  
Vous devez générer une **clé API** pour utiliser ce service.

**Procédure :**  
1. Rendez-vous sur [https://console.curseforge.com](https://console.curseforge.com)  
2. Créez un compte ou connectez-vous.  
3. Générez une clé API.  
4. Ajoutez cette clé dans les **paramètres de l’extension** sur le panel.

---

# 🧩 Intégration propre des informations de l’Egg dans le Frontend Pterodactyl

Tout a été géré **proprement côté frontend**, à travers la mise à jour de **deux fichiers clés** permettant de **demander et exploiter les informations de l’Egg** associées à chaque serveur.

---

## 1. 📁 Fichier Frontend : `resources/scripts/api/server/getServer.ts`

Ce fichier est responsable de l’appel API principal servant à récupérer les données d’un serveur.  
Il a été modifié pour **enrichir la réponse avec les informations de l’Egg**.

### 🎯 Objectif
Adapter l’appel API pour inclure systématiquement les données de l’Egg et les rendre disponibles pour l’ensemble de l’application.

---

### 🔧 Modifications Apportées

### ✅ 1. Mise à jour de la structure de données (CORRECTION 1)

L’interface Server a été mise à jour pour intégrer officiellement une nouvelle propriété egg.

```ts
export interface Server {
    // ... autres propriétés
    egg: {
        uuid: string;
        name: string;
    };
}
```

---

### ✅ 2. Transformation des données (CORRECTION 2)

La fonction rawDataToServerObject a été adaptée pour extraire les données de l’Egg
(depuis relationships) et les associer à la propriété egg.

```ts
egg: (data.relationships?.egg as any)?.attributes ?? { uuid: '', name: '' },
```

---

### ✅ 3. Modification de l’appel API *(CORRECTION 3)*
L’appel HTTP a été ajusté pour inclure le paramètre `?include=egg`,  
forçant ainsi l’API à renvoyer les informations de l’Egg.

```ts
http.get(`/api/client/servers/${uuid}?include=egg`)
```

---

## 🧠 Résultat

Le système de gestion d’état de Pterodactyl reçoit et stocke désormais l’UUID et le nom de l’Egg
pour chaque serveur.
Ces informations sont ainsi fiablement disponibles pour tous les autres composants frontend.

---

## 2. 🧭 Fichier Frontend : resources/scripts/routers/ServerRouter.tsx

Ce fichier gère l’affichage de la barre de navigation et du contenu des pages serveur.
Il a été mis à jour pour exploiter les données enrichies du serveur, notamment l’UUID de l’Egg.

## 🎯 Objectif

Utiliser l’UUID de l’Egg (désormais toujours présent) pour afficher ou masquer dynamiquement les onglets relatifs aux gestionnaires de plugins (Minecraft / Rust).

---

🔧 Modifications Apportées
### ✅ 1. Récupération de l’UUID de l’Egg

Une nouvelle ligne permet d’accéder à l’UUID directement depuis le store du serveur.

```tsx
const eggUuid = ServerContext.useStoreState((state) => state.server.data?.egg?.uuid);
```

---

### ✅ 2. Définition des listes d’UUIDs

Deux constantes regroupent les UUIDs des Eggs Minecraft et Rust, pour simplifier les vérifications.

```tsx
const MINECRAFT_EGG_UUIDS = [ /* ...vos UUIDs Minecraft... */ ];
```

---

### ✅ 3. Filtrage des liens de navigation

Une condition .filter() a été ajoutée pour n’afficher les onglets Plugins MC ou Rust Plugins
que lorsque l’Egg du serveur correspond à la bonne catégorie.

```tsx
.filter((route) => {
    if (route.path === '/mcplugins') {
        return MINECRAFT_EGG_UUIDS.includes(eggUuid || '');
    }
    return !!route.name;
})
```

---

## 🧠 Résultat

L’interface utilisateur devient totalement dynamique :
les onglets spécifiques aux plugins apparaissent uniquement lorsque c’est pertinent.
Aucun bug d’affichage, aucune supposition — le rendu dépend directement des données réelles du serveur.

---

## 💬 Support

- Serveur Discord : [https://discord.gg/hNXqvgFNYD](https://discord.gg/hNXqvgFNYD)
- Développeur : **@Magic Artistes**  
- Discord ID : `357614971422507009`

---

## 📜 Conditions d’Utilisation

1. Vous ne pouvez pas revendre ni redistribuer ce module.  
2. Les rétrofacturations sont strictement interdites.  
3. L’upload du plugin sur des sites tiers est interdit.  
4. Les mises à jour ne sont pas garanties.  

