# Configuration API - Guide rapide

## 🎯 Mode actuel : MOCK (100% fonctionnel sans backend)

L'application fonctionne actuellement en **mode MOCK** complet. Tous les services utilisent des données simulées.

## 🔄 Basculer vers l'API .NET 8

### Option 1 : Configuration globale

Éditez `/config/apiMode.ts` :

```typescript
export const apiConfig: ApiConfiguration = {
  defaultMode: 'api',  // ← Changez 'mock' en 'api'
  apiBaseUrl: 'http://localhost:5000/api',  // URL de votre API
  // ...
}
```

### Option 2 : Par service (recommandé)

Activez l'API progressivement pour chaque service :

```typescript
services: {
  boats: 'api',        // ✅ Utilisera ApiBoatService
  users: 'mock',       // ⏸️  Continue avec MockUserService
  bookings: 'api',     // ✅ Utilisera ApiBookingService
  auth: 'api',         // ✅ Utilisera ApiAuthService
}
```

### Option 3 : Variable d'environnement

Créez un fichier `.env` à la racine :

```env
# URL de l'API .NET 8
REACT_APP_API_URL=http://localhost:5000/api

# ou pour la production
REACT_APP_API_URL=https://api.sailingloc.com/api
```

## 🧪 Tester avec l'API locale

1. **Démarrez votre API .NET 8** sur `http://localhost:5000`

2. **Modifiez la configuration** :
   ```typescript
   apiBaseUrl: 'http://localhost:5000/api',
   services: {
     boats: 'api',  // Teste d'abord les bateaux
   }
   ```

3. **Vérifiez les logs console** :
   ```
   [ServiceFactory] BoatService initialized in API mode
   [API] boats.getBoats
   ```

4. **Si l'API n'est pas disponible**, l'app retombera sur les mocks automatiquement

## 📊 États des services

| Service | Mode actuel | Prêt pour API | Endpoints requis |
|---------|-------------|---------------|------------------|
| Boats | MOCK | ✅ Oui | `/api/boats/*` |
| Users | MOCK | ✅ Oui | `/api/users/*` |
| Bookings | MOCK | ✅ Oui | `/api/bookings/*` |
| Auth | MOCK | ✅ Oui | `/api/auth/*` |

## ⚙️ Configuration avancée

### Timeout et retry

```typescript
options: {
  timeout: 30000,       // Timeout en ms (30 secondes)
  retryAttempts: 2,     // Nombre de tentatives en cas d'échec
  enableLogging: true,  // Logs de debug en console
}
```

### CORS (à configurer côté API .NET 8)

Votre API doit autoriser les requêtes depuis `http://localhost:3000` :

```csharp
// Program.cs ou Startup.cs
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowReactApp",
        policy =>
        {
            policy.WithOrigins("http://localhost:3000")
                  .AllowAnyHeader()
                  .AllowAnyMethod();
        });
});

app.UseCors("AllowReactApp");
```

## 🔍 Debugging

### Voir les requêtes API

Ouvrez la console développeur (F12) et consultez :
- **Console** : Logs des services
- **Network** : Requêtes HTTP vers l'API

### Forcer le mode Mock

Si l'API pose problème, forcez le mode mock :

```typescript
services: {
  boats: 'mock',
  users: 'mock',
  bookings: 'mock',
  auth: 'mock',
}
```

## 📝 Checklist de migration

- [ ] API .NET 8 démarrée
- [ ] CORS configuré
- [ ] URL de base correcte dans `apiMode.ts`
- [ ] Un service activé en mode 'api'
- [ ] Tests des endpoints
- [ ] Validation des données
- [ ] Migration service par service
- [ ] Tests complets
- [ ] Passage en production

## 🆘 Problèmes fréquents

### Erreur CORS
```
Access to fetch at 'http://localhost:5000/api/boats' from origin 
'http://localhost:3000' has been blocked by CORS policy
```
**Solution** : Configurez CORS dans l'API .NET 8

### Timeout
```
[API] boats.getBoats - Error: Request timeout
```
**Solution** : Augmentez le timeout ou vérifiez que l'API répond

### 404 Not Found
```
HTTP 404: /api/boats not found
```
**Solution** : Vérifiez que les routes de l'API correspondent aux endpoints attendus

## 📖 Documentation complète

Voir `ARCHITECTURE_SERVICES.md` pour plus de détails sur l'architecture.

## 🎯 Résumé

- ✅ **Actuellement** : Tout en mode MOCK
- 🔄 **Pour tester l'API** : Changez un service à la fois en mode 'api'
- 🚀 **Pour la prod** : Configurez `apiBaseUrl` et mettez tous les services en 'api'
