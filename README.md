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
