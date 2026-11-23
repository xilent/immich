# Filter für versteckte Personen in Immich

Dieses Dokument erklärt die neuen Änderungen zum Filtern von Assets mit versteckten Personen.

## Was wurde geändert?

Ein neuer Such-Filter `hasHiddenPeople` wurde hinzugefügt, mit dem du Assets basierend auf der Präsenz von versteckten Personen filtern kannst.

### Geänderte Dateien:

1. **`server/src/dtos/search.dto.ts`**
   - Neuer Filter `hasHiddenPeople: BoolFilterSchema` zu `searchFilterBranchShape` hinzugefügt

2. **`server/src/utils/database.ts`**
   - Neue Hilfsfunktion `hiddenFaces()` hinzugefügt (filtert Assets mit versteckten Gesichtern)
   - Filter `hasHiddenPeople` zu `branchPredicates()` hinzugefügt

## Verwendung über die API

### Assets **ohne** versteckte Personen abrufen:

```bash
curl -X POST http://localhost:3001/api/search/metadata \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "size": 100,
    "filter": {
      "hasHiddenPeople": {
        "eq": false
      }
    }
  }'
```

### Assets **mit** versteckten Personen abrufen:

```bash
curl -X POST http://localhost:3001/api/search/metadata \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "size": 100,
    "filter": {
      "hasHiddenPeople": {
        "eq": true
      }
    }
  }'
```

## Filter-Kombinationen

Du kannst `hasHiddenPeople` mit anderen Filtern kombinieren:

```json
{
  "size": 100,
  "filter": {
    "hasHiddenPeople": {
      "eq": false
    },
    "hasPeople": {
      "eq": true
    },
    "isFavorite": {
      "eq": true
    }
  }
}
```

Das würde Assets zurückgeben, die:
- **Keine** versteckten Personen haben
- Sichtbare Personen haben
- Favoriten sind

## Verwendung in der Web-UI (zukünftig)

Mit diesen Änderungen können Frontend-Entwickler einfach einen neuen Filter-Button in der Galerie hinzufügen, z.B.:
- "Nur Fotos ohne versteckte Personen anzeigen"
- "Fotos mit versteckten Personen anzeigen"

## Beispiele

### Workflow: Galerie-Ansicht ohne versteckte Personen

```javascript
// In der Web-UI würde das so aussehen:
const assets = await api.searchAssets({
  size: 250,
  filter: {
    hasHiddenPeople: { eq: false }
  }
});
```

### Workflow: Smart Album mit versteckten Personen

```javascript
// Finde alle Assets mit versteckten Personen
const hiddenAssets = await api.searchAssets({
  size: 1000,
  filter: {
    hasHiddenPeople: { eq: true }
  }
});

// Erstelle ein Album dafür
const album = await api.createAlbum({
  albumName: 'Assets mit versteckten Personen',
  assetIds: hiddenAssets.items.map(a => a.id)
});
```

## Datenbank-Details

**hiddenFaces Funktion:**
```sql
-- Findet Assets mit mindestens einem versteckten Face
SELECT asset_face.* 
FROM asset_face 
WHERE asset_face.assetId = asset.id
  AND asset_face.deletedAt IS NULL
  AND asset_face.isVisible = FALSE
```

**visibleFaces Funktion (zum Vergleich):**
```sql
-- Findet Assets mit mindestens einem sichtbaren Face
SELECT asset_face.* 
FROM asset_face 
WHERE asset_face.assetId = asset.id
  AND asset_face.deletedAt IS NULL
  AND asset_face.isVisible = TRUE
```

## Update-Strategie

Bei zukünftigen Immich-Updates bleiben diese Änderungen stabil:

1. Wenn die neuen Version kompatibel ist → kein Konflikt
2. Wenn die `search.dto.ts` oder `database.ts` von Immich geändert wird → einfach merge
3. Deine Änderungen sind **minimal** und **ortslokalisiert**

### Git-Merge bei Updates:

```bash
# Update von Immich
git fetch origin
git rebase origin/main

# Falls Konflikte entstehen
git status  # zeigt Konflikte
git diff server/src/dtos/search.dto.ts
git diff server/src/utils/database.ts

# Konflikte auflösen und weitermachen
git add .
git rebase --continue
```

## Nächste Schritte (optional)

Für ein komplettes Erlebnis kannst du auch:

1. **Web-UI Update**: Einen Filter-Button in der Galerie hinzufügen
2. **Einstellung**: Ein System-Setting für "Standard-Verhalten" hinzufügen
3. **Mobile App**: Die gleichen Filter auf der Mobile-App implementieren

Brauchst du Hilfe mit einem dieser Schritte?
