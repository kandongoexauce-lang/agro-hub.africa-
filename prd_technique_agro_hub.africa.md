# PRD : Agro-Hub.africa - Plateforme Fintech & Agrotech (Kinshasa)

## 1. Architecture Technique (Stack & Infrastructure)

### Stack Technologique Préconisée
Pour assurer légèreté, scalabilité et résilience en zone à connectivité variable (RDC).

| Couche | Technologie | Justification |
| :--- | :--- | :--- |
| **Frontend** | Next.js (React) + Tailwind CSS | SSR/SSG pour le SEO et performance, PWA native pour le offline-first. |
| **Mobile** | Flutter ou PWA | Développement cross-platform rapide, excellente gestion du cache. |
| **Backend** | Node.js (NestJS) | Architecture modulaire, typage fort (TypeScript), idéal pour les microservices. |
| **Base de Données** | PostgreSQL + Redis | Relationnel robuste (ACID) pour la Fintech + Cache pour la performance. |
| **Infrastructure** | AWS (Region Cape Town) | Proximité géographique pour réduire la latence en Afrique Centrale. |

### Architecture Logicielle
Nous optons pour un **Monolithe Modulaire** au lancement, évoluant vers des **Microservices** (Domain-Driven Design) lors de la Phase 2. Cela permet d'éviter la complexité inutile du réseau au début tout en isolant les domaines métier (Fintech, Marketplace, IoT).

### Stratégie Cloud & Latence
*   **Edge Computing :** Utilisation de Cloudfront (CDN) avec des points de présence (PoP) africains pour les assets statiques.
*   **Infrastructure :** Kubernetes (EKS) pour l'orchestration, permettant une montée en charge automatique lors des pics de récoltes/commandes.

---

## 2. Modélisation de la Base de Données

### Schéma Relationnel (Tables Clés)

```sql
-- Gestion Multi-acteurs
CREATE TABLE users (
    id UUID PRIMARY KEY,
    role VARCHAR(20), -- 'producer', 'consumer', 'b2b_buyer', 'carrier'
    phone_number VARCHAR(15) UNIQUE, -- Login principal (Mobile-first)
    is_verified BOOLEAN DEFAULT false
);

-- Marketplace & Inventaire
CREATE TABLE products (
    id UUID PRIMARY KEY,
    name_i18n JSONB, -- { 'fr': 'Manioc', 'ln': 'Kwanga' }
    category VARCHAR(50),
    stock_quantity DECIMAL,
    producer_id UUID REFERENCES users(id)
);

-- Fintech & Transactions
CREATE TABLE wallets (
    id UUID PRIMARY KEY,
    user_id UUID REFERENCES users(id),
    balance_cdf DECIMAL DEFAULT 0,
    currency VARCHAR(3) DEFAULT 'CDF'
);

CREATE TABLE transactions (
    id UUID PRIMARY KEY,
    wallet_id UUID REFERENCES wallets(id),
    amount DECIMAL,
    type VARCHAR(20), -- 'deposit', 'payment', 'withdrawal'
    status VARCHAR(20), -- 'pending', 'completed', 'failed'
    provider_ref TEXT -- Référence Mobile Money (M-Pesa, Orange Money)
);
```

---

## 3. Spécifications Fonctionnelles & UX

### Parcours Utilisateurs
*   **Producteur :** Dashboard simplifié -> Mise en ligne de récolte -> Demande de location de tracteur (Farm-as-a-Service) -> Réception paiement sur portefeuille Agro-Hub.
*   **Acheteur B2B :** Catalogue groupé -> Négociation de volume -> Paiement sécurisé -> Suivi logistique en temps réel.

### Mécanismes Spécifiques
*   **Farm-as-a-Service :** Module de réservation basé sur la géolocalisation et la disponibilité du matériel.
*   **Offline-First :** Utilisation de *IndexedDB* pour permettre aux producteurs de saisir leurs données aux champs sans connexion; synchronisation automatique dès détection du réseau.

---

## 4. Sécurité & Compliance

*   **Security-by-Design :** Validation stricte des entrées via DTO (Data Transfer Objects) dans NestJS pour contrer les injections SQL/XSS.
*   **Fintech :** Chiffrement AES-256 des données sensibles. Intégration via des agrégateurs certifiés PCI-DSS pour éviter le stockage local des numéros de carte/comptes.
*   **RBAC (Role-Based Access Control) :** Utilisation de JWT avec claims spécifiques pour restreindre l'accès aux APIs sensibles (ex: validation de transaction).

---

## 5. Roadmap de Développement (MVP)

### Phase 1 : Fondations (Mois 1-4)
*   Déploiement du noyau Marketplace (Kinshasa).
*   Intégration Mobile Money (M-Pesa, Airtel Money, Orange Money).
*   PWA de base avec mode dégradé (Offline).

### Phase 2 : Industrialisation (Mois 5-8)
*   Module de location de matériel (Mécanisation).
*   Portail B2B avec facturation automatisée.
*   Système de suivi logistique basique.

### Phase 3 : Expansion & Intelligence (Mois 9+)
*   Déploiement IoT (capteurs humidité/température intégrés).
*   Module Edtech (vidéos de formation agricole en streaming adaptatif).
*   Internationalisation (Support de 10 langues dont Lingala, Swahili, Kikongo, Tshiluba).

---

## Annexe : Exemple d'API (NestJS Controller)

```typescript
@Controller('v1/marketplace')
export class ProductsController {
  constructor(private readonly productService: ProductService) {}

  @Post('add-crop')
  @UseGuards(RolesGuard)
  @Roles('producer')
  async addCrop(@Body() createProductDto: CreateProductDto) {
    // Logique d'ajout avec gestion i18n et validation
    return this.productService.create(createProductDto);
  }
}
```
