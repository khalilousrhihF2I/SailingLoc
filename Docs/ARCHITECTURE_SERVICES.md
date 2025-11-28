# Architecture des Services - SailingLoc

## 📋 Vue d'ensemble

Ce document décrit l'architecture des services API mise en place dans l'application SailingLoc. Le système permet de basculer facilement entre des services **mock** (pour le développement sans backend) et des services **API réels** (connectés à l'API .NET 8).

## 🏗️ Structure des dossiers

```
/
├── config/
│   └── apiMode.ts              # Configuration centrale API/Mock
├── lib/
│   └── apiClient.ts            # Client HTTP pour l'API
├── services/
│   ├── interfaces/             # Interfaces (contrats des services)
│   │   ├── IBoatService.ts
│   │   ├── IUserService.ts
│   │   ├── IBookingService.ts
│   │   └── IAuthService.ts
│   ├── mock/                   # Implémentations Mock
│   │   ├── MockBoatService.ts
│   │   ├── MockUserService.ts
│   │   ├── MockBookingService.ts
│   │   └── MockAuthService.ts
│   ├── api/                    # Implémentations API réelles
│   │   ├── ApiBoatService.ts
│   │   ├── ApiUserService.ts
│   │   ├── ApiBookingService.ts
│   │   └── ApiAuthService.ts
│   └── ServiceFactory.ts       # Factory pour instancier les services
├── hooks/
│   └── useServices.ts          # Hooks React pour utiliser les services
└── data/
    └── mockData.ts             # Données mockées
```

## ⚙️ Configuration

### Fichier `/config/apiMode.ts`

C'est le **cœur de la configuration**. Il définit quel mode utiliser pour chaque service.

```typescript
export const apiConfig: ApiConfiguration = {
  defaultMode: 'mock',  // 'mock' ou 'api'
  apiBaseUrl: 'http://localhost:5000/api',
  
  services: {
    boats: 'mock',       // Utilise MockBoatService
    users: 'mock',       // Utilise MockUserService
    bookings: 'mock',    // Utilise MockBookingService
    destinations: 'mock',
    reviews: 'mock',
    auth: 'mock',        // Utilise MockAuthService
  },
  
  options: {
    timeout: 30000,
    retryAttempts: 2,
    enableLogging: true,
  }
};
```

### Basculer en mode API

Pour utiliser l'API .NET 8 au lieu des mocks :

**Option 1 : Par service**
```typescript
services: {
  boats: 'api',        // Utilise ApiBoatService
  users: 'mock',       // Continue avec MockUserService
  bookings: 'api',     // Utilise ApiBookingService
  auth: 'api',         // Utilise ApiAuthService
}
```

**Option 2 : Mode global**
```typescript
defaultMode: 'api',    // Tous les services en mode API
```

**Option 3 : Variable d'environnement**
```bash
# .env
REACT_APP_API_URL=https://api.sailingloc.com/api
```

## 🎯 Utilisation dans les composants

### Méthode 1 : Hooks React (Recommandé)

```typescript
import { useBoats, useBoat, useAuth } from '../hooks/useServices';

function SearchPage() {
  // Charger tous les bateaux avec filtres
  const { boats, loading, error } = useBoats({ 
    destination: 'Côte d\'Azur',
    priceMax: 500 
  });

  if (loading) return <div>Chargement...</div>;
  if (error) return <div>Erreur : {error}</div>;

  return (
    <div>
      {boats.map(boat => <BoatCard key={boat.id} boat={boat} />)}
    </div>
  );
}

function BoatDetailPage({ boatId }: { boatId: number }) {
  // Charger un bateau spécifique
  const { boat, loading } = useBoat(boatId);

  if (loading || !boat) return <div>Chargement...</div>;

  return <div>{boat.name}</div>;
}

function LoginPage() {
  const { login, isAuthenticated, currentUser } = useAuth();

  const handleLogin = async () => {
    const result = await login('user@example.com', 'password');
    if (result.success) {
      console.log('Connecté !', currentUser);
    }
  };

  return <button onClick={handleLogin}>Se connecter</button>;
}
```

### Méthode 2 : Services directs

```typescript
import { boatService, bookingService, authService } from '../services/ServiceFactory';

async function loadBoats() {
  try {
    // Récupérer tous les bateaux
    const boats = await boatService.getBoats();
    
    // Avec filtres
    const filteredBoats = await boatService.getBoats({
      destination: 'Grèce',
      type: 'catamaran',
      priceMin: 200,
      priceMax: 600
    });
    
    // Un bateau spécifique
    const boat = await boatService.getBoatById(1);
    
    // Créer un nouveau bateau
    const newBoat = await boatService.createBoat({
      name: 'Mon bateau',
      type: 'sailboat',
      location: 'Nice',
      // ... autres propriétés
    });
    
  } catch (error) {
    console.error('Erreur:', error);
  }
}

async function createBooking(boatId: number) {
  const booking = await bookingService.createBooking({
    boatId,
    startDate: '2025-06-01',
    endDate: '2025-06-08',
    totalPrice: 2100,
    serviceFee: 210,
    renterId: 101,
    renterName: 'John Doe',
    renterEmail: 'john@example.com'
  });
  
  console.log('Réservation créée:', booking.id);
}

async function handleAuth() {
  // Connexion
  const loginResult = await authService.login({
    email: 'user@example.com',
    password: 'password123'
  });
  
  if (loginResult.success) {
    console.log('Token:', loginResult.token);
    console.log('User:', loginResult.user);
  }
  
  // Inscription
  const registerResult = await authService.register({
    name: 'John Doe',
    email: 'john@example.com',
    password: 'password123',
    phone: '+33612345678',
    type: 'renter'
  });
  
  // Vérifier l'authentification
  const isAuth = await authService.isAuthenticated();
  
  // Utilisateur actuel
  const user = await authService.getCurrentUser();
  
  // Déconnexion
  await authService.logout();
}
```

## 📡 Endpoints API .NET 8

Les services API communiquent avec les endpoints suivants :

### Boats
```
GET    /api/boats                    # Liste des bateaux (avec filtres)
GET    /api/boats/:id                # Détails d'un bateau
POST   /api/boats                    # Créer un bateau
PUT    /api/boats/:id                # Modifier un bateau
DELETE /api/boats/:id                # Supprimer un bateau
GET    /api/boats/owner/:ownerId     # Bateaux d'un propriétaire
```

### Users
```
GET    /api/users                    # Liste des utilisateurs
GET    /api/users/:id                # Détails d'un utilisateur
GET    /api/users/email/:email       # Utilisateur par email
POST   /api/users                    # Créer un utilisateur
PUT    /api/users/:id                # Modifier un utilisateur
DELETE /api/users/:id                # Supprimer un utilisateur
PATCH  /api/users/:id/verify         # Vérifier un utilisateur
```

### Bookings
```
GET    /api/bookings                 # Liste des réservations (avec filtres)
GET    /api/bookings/:id             # Détails d'une réservation
POST   /api/bookings                 # Créer une réservation
PUT    /api/bookings/:id             # Modifier une réservation
PATCH  /api/bookings/:id/cancel      # Annuler une réservation
GET    /api/bookings/renter/:id      # Réservations d'un locataire
GET    /api/bookings/owner/:id       # Réservations d'un propriétaire
```

### Auth
```
POST   /api/auth/login               # Connexion
POST   /api/auth/register            # Inscription
POST   /api/auth/logout              # Déconnexion
GET    /api/auth/validate            # Valider le token
```

## 🔧 Fonctionnalités avancées

### Gestion des erreurs

Les services API incluent :
- ✅ Retry automatique (2 tentatives par défaut)
- ✅ Timeout configurable (30s par défaut)
- ✅ Gestion des erreurs HTTP
- ✅ Logs de debug

### Logging

Tous les appels aux services sont loggés en console :

```
[MOCK] boats.getBoats { destination: 'Côte d\'Azur' }
[API] bookings.createBooking { boatId: 1, ... }
```

Désactiver les logs :
```typescript
options: {
  enableLogging: false,
}
```

### Réinitialisation des services

```typescript
import { ServiceFactory } from '../services/ServiceFactory';

const factory = ServiceFactory.getInstance();

// Réinitialiser tous les services
factory.reset();

// Réinitialiser un service spécifique
factory.resetService('boats');
```

## 🎭 Services Mock

Les services mock simulent :
- ✅ Délais réseau réalistes (200-600ms)
- ✅ Données mockées cohérentes
- ✅ Filtrage et pagination
- ✅ Validation des données
- ✅ Erreurs simulées

Comptes de test disponibles :
```typescript
// Admin
email: 'admin@sailingloc.com'
password: 'admin123'

// Propriétaire
email: 'owner@example.com'
password: 'demo123'

// Locataire
email: 'renter@example.com'
password: 'demo123'
```

## 🚀 Migration vers l'API

### Étape 1 : Tester en mode mock
```typescript
services: {
  boats: 'mock',
  users: 'mock',
  bookings: 'mock',
  auth: 'mock',
}
```

### Étape 2 : Activer l'API service par service
```typescript
services: {
  boats: 'api',        // ✅ API prête
  users: 'mock',       // En développement
  bookings: 'mock',
  auth: 'mock',
}
```

### Étape 3 : Mode API complet
```typescript
services: {
  boats: 'api',
  users: 'api',
  bookings: 'api',
  auth: 'api',
}
```

## ✅ Avantages de cette architecture

1. **Développement indépendant** : Le frontend peut être développé sans attendre le backend
2. **Tests facilités** : Les mocks permettent de tester toutes les fonctionnalités
3. **Transition douce** : Passage progressif de mock à API, service par service
4. **Maintenabilité** : Interfaces clairement définies
5. **Flexibilité** : Configuration centralisée, facile à modifier
6. **Type-safe** : TypeScript garantit la cohérence entre mock et API
7. **Réutilisable** : Les hooks simplifient l'utilisation dans les composants

## 📝 Exemple complet

Voir `/pages/SearchPage.tsx` pour un exemple d'utilisation avec les hooks.
Voir `/pages/BookingFlow.tsx` pour un exemple d'utilisation avec les services directs.

## 🔗 Ressources

- Configuration : `/config/apiMode.ts`
- Factory : `/services/ServiceFactory.ts`
- Hooks : `/hooks/useServices.ts`
- Client HTTP : `/lib/apiClient.ts`
