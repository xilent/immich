# Installation: Filter für versteckte Personen

Diese Anleitung zeigt dir, wie du die Änderungen in deine Immich-Instanz integrierst.

## ✅ Was wurde geändert?

Nur **2 Dateien** wurden geändert:
- `server/src/dtos/search.dto.ts` (1 Zeile hinzugefügt)
- `server/src/utils/database.ts` (4 Zeilen hinzugefügt)

## 📦 Installation

### Option 1: Direkt kopieren (für selbst-gehostete Instanzen)

Wenn du deine Immich-Quelle selbst kompilierst:

```bash
# 1. Die Dateien herunterladen/kopieren
# (Die Änderungen sind minimal und sind bereits in diesem Repo vorhanden)

# 2. Neu bauen
docker-compose down
docker-compose up --build

# 3. Fertig!
```

### Option 2: Mit Git (empfohlen)

```bash
# Diese Branch mit den Änderungen checken
git checkout main
git pull origin main

# Oder: Wenn du bereits deine eigenen Änderungen hast
git rebase origin/main
# (Falls Konflikte: siehe "Update-Konflikt beheben" unten)
```

## 🧪 Test

Nach dem Neu-Start, teste die neue API:

```bash
# 1. API-Token holen (aus Immich Settings)
# 2. Anfrage testen:

curl -X POST http://localhost:3001/api/search/metadata \
  -H "Content-Type: application/json" \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -d '{
    "size": 10,
    "filter": {
      "hasHiddenPeople": {
        "eq": false
      }
    }
  }'
```

**Erfolg** = JSON-Array mit Assets ohne versteckte Personen ✅

## 🔄 Bei Immich-Updates

### Wenn keine Konflikte:

```bash
git fetch origin
git rebase origin/main
docker-compose down
docker-compose up --build
```

### Bei Konflikten:

```bash
git fetch origin
git rebase origin/main

# Git zeigt Konflikte
git status

# Optionen:
# A) Deine Änderungen behalten + neue Features:
git rebase --continue

# B) Konflikt manuell lösen:
nano server/src/dtos/search.dto.ts
# ... Änderungen überprüfen, dann:
git add server/src/dtos/search.dto.ts
git rebase --continue
```

## 🗂️ Dateiänderungen Details

### `server/src/dtos/search.dto.ts`

```diff
const searchFilterBranchShape = {
  ...
  hasPeople: BoolFilterSchema,
+ hasHiddenPeople: BoolFilterSchema,
  hasTags: BoolFilterSchema,
  ...
};
```

### `server/src/utils/database.ts`

```diff
const visibleFaces = (eb: AssetExpressionBuilder) => ...

+ const hiddenFaces = (eb: AssetExpressionBuilder) =>
+   eb
+     .selectFrom('asset_face')
+     .whereRef('asset_face.assetId', '=', 'asset.id')
+     .where('asset_face.deletedAt', 'is', null)
+     .where('asset_face.isVisible', '=', false);

...

function branchPredicates(eb: AssetExpressionBuilder, branch: SearchFilterBranch) {
  ...
  ...existsPredicates(eb, branch.hasPeople, () => visibleFaces(eb)),
+ ...existsPredicates(eb, branch.hasHiddenPeople, () => hiddenFaces(eb)),
  ...existsPredicates(eb, branch.hasTags, () => tagAssets(eb)),
  ...
}
```

## ❓ FAQ

**F: Brauche ich die Web-UI anzupassen?**
A: Nein! Die API funktioniert sofort. Wenn du einen UI-Button haben möchtest, können wir das später hinzufügen.

**F: Funktioniert das mit Docker?**
A: Ja! Docker baut automatisch neu, wenn du Dateien änderst.

**F: Was ist wenn ich diesen Filter nicht brauche?**
A: Die Änderungen sind völlig sicher. Sie beeinflussen die Galerie nicht, wenn du sie nicht verwendest.

**F: Kann ich rückgängig machen?**
A: Ja, einfach die zwei Zeilen löschen und neu bauen:
```bash
# Im Editor, die neuen Zeilen löschen
git checkout -- server/src/dtos/search.dto.ts
git checkout -- server/src/utils/database.ts
docker-compose down
docker-compose up --build
```

## 📞 Probleme?

Falls etwas nicht funktioniert:

1. **Build fehlgeschlagen?** → Logs anschauen:
```bash
docker-compose logs server
```

2. **Anfrage gibt Fehler?** → API-Token überprüfen:
```bash
# Im Immich Settings kopieren und einfügen
```

3. **Filter funktioniert nicht?** → Stelle sicher, dass:
   - Du Assets mit versteckten Personen hast
   - Die API-Antwort JSON ist (nicht HTML-Fehler)

## 📚 Weitere Ressourcen

- [Filter-Dokumentation](./FILTER_HIDDEN_PEOPLE.md)
- [Immich API-Dokumentation](http://localhost:3001/api)
- [Immich GitHub](https://github.com/immich-app/immich)

Viel Erfolg! 🚀
