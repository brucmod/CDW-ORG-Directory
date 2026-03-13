# CDW Sales Org Directory

Interactive directory for the CDW Sales org — searchable, filterable, with table / card / group-by-manager views.

## Files

| File | Purpose |
|------|---------|
| `index.html` | App shell — all UI, logic, and styles |
| `data.json` | Coworker records (Sales sector only, Unknown title groups excluded) |
| `README.md` | This file |

## Updating the data

**Option A — In-app upload (easiest)**
1. Open the live site
2. Click **↑ Update Data** in the filter bar
3. Drop in the new `.xlsx` export — the app re-processes and reloads instantly
4. This only updates your local browser session. To persist the update, use Option B.

**Option B — Replace data.json (permanent)**
1. Run the data export script (or use the Python snippet below) on the new `.xlsx`
2. Replace `data.json` in the repo
3. Commit and push — GitHub Pages picks it up automatically

### Quick Python snippet to regenerate data.json

```python
import pandas as pd, json, re

df = pd.read_excel('CDW_ORG.xlsx').fillna('')
sales = df[df['Sector'] == 'Sales']

def extract_state(loc):
    m = re.search(r'VIRTUAL - ([A-Z]{2})\)', str(loc), re.I)
    if m: return m.group(1).upper()
    m = re.search(r' - ([A-Z]{2}) \(', str(loc))
    if m: return m.group(1).upper()
    return ''

def clean(s):
    return str(s).strip()

def clean_loc(s):
    return re.sub(r'\s*\([^)]*\)\s*$', '', str(s)).strip().rstrip(' -')

def title(s):
    return str(s).strip().title()

def ref_name(s):
    return title(re.sub(r'\s*\([^)]*\)\s*$', '', str(s)).strip())

def person_name(s):
    return title(re.sub(r'^\([^)]*\)\s*', '', str(s)).strip())

records = []
for _, r in sales.iterrows():
    name = person_name(r['CoworkerName'])
    if not name: continue
    if clean(r['CoworkerTitleGroupDescription']) == 'Unknown': continue
    los = str(r['LOSMonth']).strip()
    try: los_disp = f"{int(float(los))} months"
    except: los_disp = ''
    records.append({
        "name": name, "email": clean(r['EmailAddress']),
        "phone": clean(r['DirectPhone']),
        "location": clean_loc(r['CoworkerLocationDescription']),
        "state": extract_state(r['CoworkerLocationDescription']),
        "title": clean(r['CoworkerTitleDescription']),
        "titleGroup": clean(r['CoworkerTitleGroupDescription']),
        "losMonths": los_disp, "losGroup": clean(r['LOSGroupDescription']),
        "academyFlag": clean(r['academyamflagdescription']),
        "manager": ref_name(r['Manager']), "managerEmail": clean(r['ManagerEmailAddress']),
        "managerTitle": clean(r['ManagerTitle']),
        "director": ref_name(r['Director']), "directorEmail": clean(r['Director Email']),
        "directorTitle": clean(r['Director Title']),
        "segment": clean(r['Segment']), "channel": clean(r['Channel']),
        "tier": clean(r['Tier']), "region": clean(r['Region']),
        "area": clean(r['Area']), "district": clean(r['District']),
    })

with open('data.json', 'w') as f:
    json.dump(records, f, indent=2)

print(f"Written {len(records)} records to data.json")
```

## Hosting on GitHub Pages

1. Create a new GitHub repo (public or private with Pages enabled)
2. Push `index.html`, `data.json`, and `README.md`
3. Go to **Settings → Pages → Source** and set branch to `main`, folder to `/ (root)`
4. Your site will be live at `https://<your-org>.github.io/<repo-name>/`

## Filters

| Filter | Field |
|--------|-------|
| Segment | Commercial / Government / Education |
| Channel | Corporate, Federal, K-12, Healthcare, etc. |
| Tier | Majors, Territory, Enterprise, etc. |
| Region | East, West, Central, etc. |
| District | Granular team grouping |
| Manager | Direct manager name |

## Views

- **Table** — sortable columns, click any row for full details
- **Cards** — visual grid with segment / channel / tier / district tags
- **Groups** — collapsed/expandable sections grouped by manager
