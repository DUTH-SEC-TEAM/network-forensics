# GitHub setup

## Προαπαιτούμενα

- [ ] Λογαριασμός GitHub (αν δεν έχεις: [github.com/signup](https://github.com/signup))
- [ ] Πρόσκληση στον οργανισμό DUTH-SEC-TEAM (από τον team leader)
- [ ] Εγκατεστημένο [Obsidian](https://obsidian.md)
- [ ] Εγκατεστημένο Git

## Βήμα 1: Εγκατάσταση Git

### Windows

1. Κατέβασε το Git από το [git-scm.com](https://git-scm.com)
2. Εγκατάστησε με **default ρυθμίσεις**
3. Χρησιμοποίησε **Git Bash** για τα commands.

Επιβεβαίωσε την εγκατάσταση:

```bash
git --version
```

### Linux (Debian/Ubuntu)

```bash
sudo apt install git
```

### MacOS

```bash
brew install git
```

## Βήμα 2: Δημιουργία SSH Key

> [!important] Αν έχεις ήδη προσωπικό SSH key για το GitHub, **μην το χρησιμοποιήσεις εδώ**. Δημιούργησε ένα ξεχωριστό key αποκλειστικά για τον οργανισμό DUTH-SEC-TEAM, όπως δείχνει το βήμα παρακάτω. Στο επόμενο βήμα (SSH Config) θα ρυθμίσουμε το σύστημα ώστε να χρησιμοποιεί το σωστό key για κάθε περίπτωση, χωρίς να μπερδεύονται τα δύο.

### Linux / Mac

```bash
ssh-keygen -t ed25519 -C "το_email_σου@gmail.com" -f ~/.ssh/id_ed25519_github_team
```

- Όταν ρωτάει **passphrase**: πάτα Enter (κενό) Είναι σαν κωδικός που θα χρειάζεται να βάζεις κάθε φορά που κάνεις push/pull (αλληλεπιδράς με το github)

Αντέγραψε το public key:

```bash
cat ~/.ssh/id_ed25519_github_team.pub
```

### Windows

```bash
# Git Bash (δουλεύει ως έχει)
ssh-keygen -t ed25519 -C "to_email_sou@gmail.com" -f ~/.ssh/id_ed25519_github_team

# PowerShell (αν θες να το τρέξεις εκεί)
ssh-keygen -t ed25519 -C "to_email_sou@gmail.com" -f "$HOME\.ssh\id_ed25519_github_team"
```

- Όταν ρωτάει **passphrase**: πάτα Enter (κενό) Είναι σαν κωδικός που θα χρειάζεται να βάζεις κάθε φορά που κάνεις push/pull (αλληλεπιδράς με το github)

Αντέγραψε το public key:

```bash
# Git Bash
cat ~/.ssh/id_ed25519_github_team.pub

# PowerShell
Get-Content "$HOME\.ssh\id_ed25519_github_team.pub"
```

## Βήμα 3: Ρύθμιση SSH Config

Επειδή το key δεν έχει το προεπιλεγμένο όνομα (και επειδή μπορεί να έχεις ήδη κάποιο **άλλο** προσωπικό key για το GitHub), πρέπει να πούμε ρητά στο SSH ποιο key να χρησιμοποιεί όταν δουλεύεις με τον οργανισμό DUTH-SEC-TEAM — χωρίς να πειράξουμε τυχόν υπάρχουσα προσωπική σου ρύθμιση.

Αυτό γίνεται με ένα **ξεχωριστό host alias**: ένα ψευδώνυμο που το SSH αναγνωρίζει σαν να είναι hostname, αλλά στην ουσία δείχνει στο `github.com` — χρησιμοποιώντας όμως το δικό του συγκεκριμένο key.

### Linux / Mac

Άνοιξε (ή δημιούργησε αν δεν υπάρχει) το αρχείο `~/.ssh/config`:

```bash
nano ~/.ssh/config
```

Πρόσθεσε στο τέλος:

```
Host github-duth-team
    HostName github.com
    User git
    IdentityFile ~/.ssh/id_ed25519_github_team
    IdentitiesOnly yes
```

### Windows

Άνοιξε (ή δημιούργησε) το αρχείο `C:\Users\Username\.ssh\config` (π.χ. με Notepad):

```
Host github-duth-team
    HostName github.com
    User git
    IdentityFile C:/Users/Username/.ssh/id_ed25519_github_team
    IdentitiesOnly yes
```

> [!warning] Χρησιμοποίησε **forward slashes** (`/`) στο path, ακόμα και στα Windows.

> [!important] Αν έχεις **ήδη** ένα `Host github.com` block στο config σου (π.χ. από προσωπικό key), μην το πειράξεις — άφησέ το ως έχει και απλά πρόσθεσε το `Host github-duth-team` block ως ξεχωριστή, νέα καταχώρηση.

Από εδώ και πέρα, **παντού** που θα αναφέρεται `github.com` σε σχέση με τον οργανισμό (clone URLs, remote URLs), θα χρησιμοποιούμε το alias `github-duth-team` αντί για το `github.com`.

## Βήμα 4: Προσθήκη SSH Key στο GitHub

1. Πήγαινε στο **GitHub → User navigation Menu (το icon σου πάνω δεξιά) → Settings → SSH and GPG keys**
2. Πάτα **New SSH key**
3. **Title:** `DUTH-TEAM`
4. **Key type:** `Authentication Key`
5. **Key:** επικόλλησε το output από το Βήμα 2
6. Πάτα **Add SSH key**

### Επαλήθευση σύνδεσης

Χρησιμοποίησε το **alias** που ρύθμισες στο Βήμα 3 — δεν χρειάζεται πια `-i` flag, αφού το config το χειρίζεται αυτόματα:

```bash
ssh -T git@github-duth-team
```

**Σωστό αποτέλεσμα:** `Hi [username]! You've successfully authenticated, but GitHub does not provide shell access.`

> [!warning] Αν δοκιμάσεις `ssh -T git@github.com` (χωρίς το alias), το SSH θα προσπαθήσει με το default/προσωπικό σου key, όχι με το key του οργανισμού — και πιθανόν να αποτύχει ή να συνδεθεί με λάθος λογαριασμό. Χρησιμοποίησε πάντα το alias για δουλειά με το DUTH-SEC-TEAM.

## Βήμα 5: Clone του Repository

Δημιούργησε έναν φάκελο για το vault και κάνε clone, χρησιμοποιώντας το alias:

> [!warning] Τα paths είναι ενδεικτικά — προσαρμόστε τα ανάλογα με τη δομή φακέλων του συστήματός σας.

### Windows (PowerShell)

```powershell
# Αντικατέστησε το REPO_NAME με το repo της ομάδας σου:
# network-forensics / vulnerability-exploitation / web-security

New-Item -ItemType Directory -Force -Path C:\Users\Username\DUTH-TEAM
cd C:\Users\Username\DUTH-TEAM
git clone git@github-duth-team:DUTH-SEC-TEAM/REPO_NAME.git
cd REPO_NAME
```

### Linux / Mac

```bash
# Αντικατέστησε το REPO_NAME με το repo της ομάδας σου:
# network-forensics / vulnerability-exploitation / web-security

mkdir -p ~/DUTH-TEAM
cd ~/DUTH-TEAM
git clone git@github-duth-team:DUTH-SEC-TEAM/REPO_NAME.git
cd REPO_NAME
```

> [!important] Παρατήρησε: μόνο το κομμάτι πριν το `:` άλλαξε (`github.com` → `github-duth-team`). Το path του repo (`DUTH-SEC-TEAM/REPO_NAME.git`) μένει ίδιο.

## Βήμα 6: Βασικές Ρυθμίσεις Git

### Τα στοιχεία σου

#### Αν θέλετε να ρυθμίσετε τα στοιχεία σας μια φορά:

```bash
git config --global user.name "Το Όνομά σου"
git config --global user.email "email@example.com"
```

> Έτσι δηλώνετε user_name & email για κάθε git σύνδεση που θα κάνετε σε οποιοδήποτε repository (σημείο στο σύστημα σας).

#### Αν θέλετε να ρυθμίσετε τα στοιχεία σας μόνο για το συγκεκριμένο repository:

```bash
git config user.name "Το Όνομά σου"
git config user.email "email@example.com"
```

> Έτσι δηλώνετε user_name & email μόνο για την σύνδεση στο συγκεκριμένο repository.

### Black list .obsidian στο repo

```shell
echo ".obsidian/" >> .gitignore   #.obsidian = local vault settings
git add .gitignore
git commit -m "Add .obsidian to gitignore"
git push origin main
```

> Προσθέτοντας το `.obsidian` στο `.gitignore` λες στο **github** όταν θα κάνω push περιεχόμενο μην λάβεις υπόψη το `.obsidian`

## Βήμα 7: Άνοιγμα ως Obsidian Vault

1. Άνοιξε το **Obsidian**
2. **Open another vault → Open folder as vault**
3. Επιλογή Φακέλου
    - Windows -> `C:\Users\Username\DUTH-TEAM\REPO_NAME`
    - Linux / Mac -> `~/DUTH-TEAM/REPO_NAME`

## Βήμα 8: Εγκατάσταση Git Plugin

1. **Settings → Community plugins → Turn on community plugins**
2. **Browse → αναζήτησε "Git" → Install → Enable**
3. Ρυθμίσεις plugin:

|Ρύθμιση|Τιμή|
|---|---|
|Auto pull interval|`10`|
|Pull on startup|On|
|Push on commit-and-sync|On|
|Pull on commit-and-sync|On|
|Author name|Το όνομά σου|
|Author email|Το email σου|
|Custom Git binary path|Linux/Mac ->`/usr/bin/git` <br>Windows ->`C:\Program Files\Git\bin\git.exe`|

> [!important] Το Obsidian Git plugin δεν χρειάζεται καμία επιπλέον ρύθμιση για το SSH key — διαβάζει αυτόματα το `origin` URL που ήδη ρυθμίστηκε σωστά στο Βήμα 5 (με το alias `github-duth-team`), οπότε λειτουργεί διαφανώς.

# Workflows

## Ανάγνωση αλλαγών (Pull)

Οι αλλαγές κατεβαίνουν **αυτόματα κάθε 10 λεπτά**. Για άμεση ενημέρωση:

`Ctrl+P` → `Git: Pull`

---

## Πρόταση αλλαγής (Push → PR)

### Βήμα 1: Δημιούργησε νέο branch

> **Γιατί;** Ποτέ μην δουλεύεις απευθείας στο `main`. Κάθε αλλαγή γίνεται σε ξεχωριστό branch.

`Ctrl+P` → `Git: Create new branch` → δώσε περιγραφικό όνομα

Παραδείγματα καλών ονομάτων:

- `fix/typo-metasploitable2-tool`
- `add/new-tool-nmap`
- `update/readme`

### Βήμα 2: Κάνε τις αλλαγές σου

Επεξεργάσου τα notes κανονικά στο Obsidian.

### Βήμα 3: Commit

`Ctrl+P` → `Git: Commit all changes` → γράψε περιγραφικό μήνυμα

Παραδείγματα καλών μηνυμάτων:

- `Διόρθωση τυπογραφικών λαθών`
- `Προσθήκη οδηγιών εγκατάστασης και χρήσης εργαλείου nmap`
- `Ενημέρωση του readme file - Προσθήκη link εργαλείου`

### Βήμα 4: Push

`Ctrl+P` → `Git: Push` → επέλεξε `origin` → γράψε το όνομα του branch σου (βήμα 1)

### Βήμα 5: Pull Request στο GitHub

1. Πήγαινε στο github.com/DUTH-SEC-TEAM/REPO_NAME
2. **Branches** → βρες το branch σου → **New pull request**
3. Γράψε σύντομη περιγραφή των αλλαγών
4. **Create pull request**
5. Περίμενε έγκριση από τον Team Leader

### Βήμα 6: Μετά την έγκριση — Καθάρισε τα branches

Αν το ίδιο branch δεν χρειάζεται πλέον, δεν είναι σχετικό της επόμενης αλλαγής μπορείς να το καθαρίσεις από το local repository σου.

```bash
git checkout main
git pull
git branch -d BRANCH_NAME
git remote prune origin
```

# Διαχείριση Branches

## Δες τα branches σου

```bash
git branch -a
```

- Χωρίς πρόθεμα = local branches
- `remotes/origin/` = remote branches στο GitHub

## Διέγραψε branch (μετά από merge)

```bash
git branch -d BRANCH_NAME
```

## Καθάρισε παλιά remote branches

```bash
git remote prune origin
```

# Λυμένα προβλήματα

## "Your local changes would be overwritten"

Έχεις αλλαγές που δεν έχεις κάνει commit. Λύση:

```bash
git stash
git pull
```

## "There is no tracking information for the current branch"

```bash
git branch --set-upstream-to=origin/main main
```

## "The upstream branch does not match"

```bash
git push origin HEAD
```

## "Permission denied (publickey)" σε clone/pull/push

Πιθανότατα το SSH config (Βήμα 3) δεν έχει ρυθμιστεί σωστά, ή χρησιμοποίησες `github.com` αντί για το alias `github-duth-team` στο remote URL. Έλεγξε:

```bash
# Δες το τρέχον remote URL
git remote -v

# Αν δείχνει github.com αντί για github-duth-team, διόρθωσέ το:
git remote set-url origin git@github-duth-team:DUTH-SEC-TEAM/REPO_NAME.git
```

## Το Obsidian Git δείχνει "Git is not ready"

Βάλε το path του git στις ρυθμίσεις:

- Linux/Mac: `/usr/bin/git`
- Windows: `C:\Program Files\Git\bin\git.exe`

---

## Βοήθεια

Αν αντιμετωπίσεις πρόβλημα επικοινώνησε με τον Team Leader σου.

> [!important]
> 
> - **Ποτέ** μην κάνεις direct push στο `main`
> - Κάθε αλλαγή γίνεται μέσω **νέου branch + Pull Request**
> - **Περιγραφικά** ονόματα branches και commit messages
> - **Καθάρισε** τα παλιά branches μετά το merge
> - Οι αλλαγές εμφανίζονται αυτόματα κάθε **10 λεπτά** (auto-pull)
> - Για να δεις αλλαγές αμέσως: `Ctrl+P` → `Git: Pull`
> - **Κάνε pull** πριν ξεκινήσεις δουλειά για να έχεις τις τελευταίες αλλαγές
> - Χρησιμοποίησε πάντα το SSH alias `github-duth-team` (όχι `github.com`) για δουλειά με τον οργανισμό