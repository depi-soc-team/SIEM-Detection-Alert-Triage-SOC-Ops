# دليل Git و GitHub للتيم

دليل عملي لأي حد في التيم، حتى لو أول مرة يستخدم Git. كل خطوة مشروحة بالترتيب، والأوامر جاهزة تعمل لها copy-paste.

> القواعد الرسمية موجودة في [CONTRIBUTING.md](../CONTRIBUTING.md). الدليل ده بيشرح **إزاي** تطبقها خطوة بخطوة.

## المحتويات

1. [الإعداد لأول مرة (Windows)](#1-الإعداد-لأول-مرة-windows)
2. [الشغل اليومي — 3 طرق](#2-الشغل-اليومي--3-طرق)
3. [القواعد](#3-القواعد)
4. [الـ Pull Request](#4-الـ-pull-request)
5. [الـ Review والـ Merge: مين بيعمل إيه](#5-الـ-review-والـ-merge-مين-بيعمل-إيه)
6. [Checklist للـ Reviewer](#6-checklist-للـ-reviewer)
7. [تحريك الكروت على الـ SOC Project Board](#7-تحريك-الكروت-على-الـ-soc-project-board)
8. [مشاكل شائعة وحلها](#8-مشاكل-شائعة-وحلها)
9. [Cheat sheet](#9-cheat-sheet)

---

## 1. الإعداد لأول مرة (Windows)

بتعمل الخطوات دي **مرة واحدة بس** على جهازك.

> لو مش هتسطب حاجة خالص، اقفز على [الطريقة 3: موقع GitHub بس](#الطريقة-3-موقع-github-بس-من-غير-تسطيب). بس هتكون محدود (مفيش screenshots كتير ولا فولدرات كبيرة بسهولة).

### 1.1 سطّب Git و GitHub Desktop

1. نزّل Git من <https://git-scm.com/download/win> وسطّبه. سيب كل الاختيارات الـ default واضغط Next لحد الآخر.
2. نزّل GitHub Desktop من <https://desktop.github.com> وسطّبه.
3. افتح PowerShell واتأكد إن Git اتسطب:

```powershell
git --version
```

لو طلعلك رقم version (زي `git version 2.x`) يبقى تمام.

### 1.2 اعمل Sign in

1. افتح GitHub Desktop.
2. **File > Options > Accounts > Sign in** وادخل بأكونت GitHub بتاعك في المتصفح.
3. اتأكد إن الأكونت ده عضو في الـ org `depi-soc-team` (لو مش شايف الـ repo، كلم الـ Lead).

### 1.3 اعمل clone للـ repo في `C:\dev\`

> ⚠️ **ماتحطش الـ repo أبداً جوه OneDrive أو Desktop.**
> OneDrive بيعمل sync للملفات وهو شغال، وده بيبوّظ فولدر `.git` (ملفات locked، conflicts غريبة، وأحياناً الـ repo بيبوظ خالص). الـ Desktop على أجهزة كتير بيبقى جوه OneDrive أصلاً.

اعمل فولدر `C:\dev\` الأول:

```powershell
New-Item -ItemType Directory -Force C:\dev
```

**بـ GitHub Desktop:**

1. **File > Clone repository**
2. تاب **GitHub.com** واختار `depi-soc-team/SIEM-Detection-Alert-Triage-SOC-Ops`
3. في **Local path** اكتب: `C:\dev\SIEM-Detection-Alert-Triage-SOC-Ops`
4. اضغط **Clone**

**أو بالـ command line:**

```powershell
cd C:\dev
git clone https://github.com/depi-soc-team/SIEM-Detection-Alert-Triage-SOC-Ops.git
cd SIEM-Detection-Alert-Triage-SOC-Ops
```

### 1.4 اظبط اسمك والإيميل (GitHub noreply)

استخدم الـ **noreply email** بتاع GitHub عشان إيميلك الحقيقي مايظهرش في الـ commits.

1. على GitHub: **Settings > Emails**
2. علّم على **Keep my email addresses private**
3. هتلاقي إيميل شكله كده: `<ID>+<username>@users.noreply.github.com` — انسخه.
4. في PowerShell:

```powershell
git config --global user.name "Your Name"
git config --global user.email "<ID>+<username>@users.noreply.github.com"
```

5. اتأكد:

```powershell
git config --global --list
```

> في GitHub Desktop: **File > Options > Git** هتلاقي نفس الإعدادات، اتأكد إن الإيميل هو الـ noreply.

---

## 2. الشغل اليومي — 3 طرق

نفس الخطوات الست كل مرة، اختار الطريقة اللي تريحك:

**update main → create branch → edit → commit → push → open PR**

> في الأمثلة تحت، غيّر `<member>` لاسمك الأول (lowercase) **من غير** علامات `< >` — مثلاً `git switch -c yourname/fortigate-syslog`. PowerShell هيطلع error لو سبت `<` و `>`.

### مقارنة سريعة

| الخطوة | GitHub Desktop | Command line (PowerShell) | موقع GitHub بس |
|---|---|---|---|
| 1. update main | **Current branch** = `main` ثم **Fetch origin** ثم **Pull origin** | `git switch main` ثم `git pull` | مش محتاج — الموقع دايماً على آخر نسخة |
| 2. create branch | **Current branch > New branch** | `git switch -c <member>/fortigate-syslog` | بيتعمل وقت الـ commit (خطوة 4) |
| 3. edit | عدّل الملفات بأي editor (VS Code مثلاً) | عدّل الملفات بأي editor | **Add file > Upload files** أو أيقونة القلم ✏️ (**Edit this file**) |
| 4. commit | اكتب Summary تحت على الشمال ثم **Commit to <member>/...** | `git add <file>` ثم `git commit -m "..."` | **Commit changes...** ثم اختار **Create a new branch for this commit and start a pull request** |
| 5. push | **Publish branch** (أول مرة) أو **Push origin** | `git push -u origin <member>/fortigate-syslog` | بيحصل تلقائي |
| 6. open PR | **Preview Pull Request** ثم **Create pull request** | افتح اللينك اللي بيطلع بعد الـ push | **Propose changes** ثم **Create pull request** |

التفاصيل تحت 👇

### الطريقة 1: GitHub Desktop

1. **update main**
   - من فوق: **Current branch** واختار `main`.
   - اضغط **Fetch origin**، ولو ظهر **Pull origin** اضغطه.
2. **create branch**
   - **Current branch > New branch**
   - اكتب الاسم: `<member>/fortigate-syslog` (شوف [القواعد](#3-القواعد))
   - اتأكد إن **Create branch based on** = `main`، واضغط **Create branch**.
3. **edit**
   - اضغط **Show in Explorer** أو **Open in Visual Studio Code** وعدّل الملفات.
   - ارجع لـ GitHub Desktop، هتلاقي التغييرات في تاب **Changes**.
4. **commit**
   - راجع التغييرات في تاب **Changes** — اتأكد مفيش IP أو باسورد حقيقي.
   - تحت على الشمال: اكتب **Summary** واضح، مثلاً `Add FortiGate syslog onboarding guide`.
   - اضغط **Commit to <member>/fortigate-syslog**.
5. **push**
   - أول مرة: **Publish branch**.
   - بعد كده: **Push origin**.
6. **open PR**
   - اضغط **Preview Pull Request** ثم **Create pull request** — هيفتح المتصفح.
   - كمّل الـ PR زي [قسم 4](#4-الـ-pull-request).

### الطريقة 2: Command line (PowerShell)

افتح PowerShell جوه فولدر الـ repo:

```powershell
cd C:\dev\SIEM-Detection-Alert-Triage-SOC-Ops
```

1. **update main**

```powershell
git switch main
git pull
```

2. **create branch**

```powershell
git switch -c <member>/fortigate-syslog
```

3. **edit** — عدّل الملفات بأي editor، وبعدين شوف إيه اللي اتغير:

```powershell
git status
```

4. **commit** — ضيف الملفات اللي عايزها بس (مش `git add .` عشوائي)، وراجع قبل الـ commit:

```powershell
git add 02-onboarding/fortigate.md
git diff --staged
git commit -m "Add FortiGate syslog onboarding guide"
```

5. **push**

```powershell
git push -u origin <member>/fortigate-syslog
```

بعد أول مرة، أي commit جديد على نفس الـ branch بيكفيه:

```powershell
git push
```

6. **open PR** — بعد الـ push، Git بيطبع لينك فيه `pull/new/...`. افتحه في المتصفح واضغط **Create pull request**.
   أو روح للـ repo على GitHub وهتلاقي زرار أصفر **Compare & pull request**.

### الطريقة 3: موقع GitHub بس (من غير تسطيب)

مناسبة لتعديلات صغيرة أو رفع ملف/screenshot.

1. **update main** — مش محتاج، الموقع دايماً على آخر نسخة من `main`.
2. **edit**
   - **ملف موجود:** افتح الملف ← أيقونة القلم ✏️ **Edit this file**.
   - **ملف جديد:** ادخل الفولدر الصح ← **Add file > Create new file**. ممكن تعمل فولدر جديد لو كتبت في الاسم `images/` مثلاً.
   - **رفع ملفات (screenshots):** ادخل الفولدر الصح ← **Add file > Upload files** واسحب الملفات.
3. **commit + create branch** (مع بعض)
   - اضغط **Commit changes...**
   - اكتب **Commit message** واضح.
   - اختار **Create a new branch for this commit and start a pull request**.
   - ⚠️ غيّر اسم الـ branch المقترح (زي `<username>-patch-1`) لاسم بالقواعد: `<member>/fortigate-syslog`.
   - اضغط **Propose changes**.
4. **push** — بيحصل تلقائي.
5. **open PR** — هتتنقل لصفحة **Open a pull request**. كمّل زي [قسم 4](#4-الـ-pull-request) واضغط **Create pull request**.

> عايز تعدّل تاني على نفس الـ PR؟ من الموقع اختار الـ branch بتاعك من القايمة اللي فوق على الشمال (مكتوب فيها `main`) قبل ما تعدّل، واختار **Commit directly to the <member>/... branch**.

---

## 3. القواعد

1. **branch لكل task، مش لكل شخص.** كل issue = branch = PR. ماتعملش branch باسمك بس (زي `<member>`) وتحط فيه كل شغلك.
2. **اسم الـ branch:** `<member>/<short-task>`
   - `member`: اسمك الأول بالإنجليزي، lowercase
   - `short-task`: كلمتين تلاتة، lowercase و kebab-case (شرطة بين الكلمات)
   - أمثلة: `<member>/project-plan`, `<member>/fortigate-syslog`, `<member>/dr-001-suspicious-powershell`, `<member>/tr-001-brute-force`, `<member>/soc-dashboard`
3. **دايماً اعمل pull لـ `main` قبل ما تبدأ task جديدة.** (خطوة 1 فوق)
4. **امسح الـ branch بعد الـ merge.** على صفحة الـ PR بعد الـ merge اضغط **Delete branch**. وعلى جهازك:

```powershell
git switch main
git pull
git branch -d <member>/fortigate-syslog
```

5. **ماتعملش commit على `main` مباشرة.** لو حصل بالغلط شوف [قسم 8](#8-مشاكل-شائعة-وحلها).

---

## 4. الـ Pull Request

1. **Title:** واضح وقصير، بيوصف التغيير. مثلاً: `Add FortiGate syslog onboarding guide`
2. **Description:** استخدم الـ template ده (نفس اللي في [CONTRIBUTING.md](../CONTRIBUTING.md)):

```markdown
## Summary
What this PR adds/changes.

## Evidence
Screenshots / validation (sanitized).

Closes #N
```

3. **`Closes #N`:** غيّر `N` لرقم الـ issue (مثلاً `Closes #5`). كده الـ issue هيتقفل لوحده لما الـ PR يتعمله merge.
4. **Reviewer:** في الـ sidebar على اليمين ← **Reviewers** ⚙️ ← اختار الشخص اللي ماسك الـ **Support** role المكتوب في الـ issue.
   - لو الـ Support "All members"، اختار أي حد من التيم.
   - لو الـ Support هو انت نفسك أو مش موجود، اختار الـ Lead.
5. اضغط **Create pull request**، وحرّك الكارت لـ **Review** (شوف [قسم 7](#7-تحريك-الكروت-على-الـ-soc-project-board)).

---

## 5. الـ Review والـ Merge: مين بيعمل إيه

| الدور | مين | بيعمل إيه |
|---|---|---|
| Author | اللي ماسك الـ **Owner** role في الـ issue | بيفتح الـ PR ويطلب review ويصلّح الـ comments |
| Reviewer | اللي ماسك الـ **Support** role في الـ issue | بيراجع بالـ [checklist](#6-checklist-للـ-reviewer) ويعمل **Approve** |
| Merger | الـ Lead | بيعمل **Merge** بعد الـ approval |

**خطوات الـ Reviewer على GitHub:**

1. افتح الـ PR ← تاب **Files changed**.
2. لو عندك ملاحظة على سطر، اضغط **+** جنب السطر واكتب comment.
3. فوق على اليمين: **Review changes**
   - **Approve** لو كله تمام
   - **Request changes** لو في حاجة لازم تتصلح
4. اضغط **Submit review**.

**خطوات الـ Merge (الـ Lead):**

1. اتأكد إن في **Approve** واحد على الأقل.
2. اضغط **Merge pull request** ← **Confirm merge**.
3. اضغط **Delete branch**.

**لو الـ Lead مش متاح:** الـ reviewer نفسه يقدر يعمل merge **بعد** ما يعمل approve.

> ⚠️ **ماتعملش merge للـ PR بتاعك من غير review أبداً.**
> ده team rule — GitHub مش هيمنعك (الـ free plan مع private repo مفيهوش branch protection)، فإحنا بنعتمد على بعض.

---

## 6. Checklist للـ Reviewer

قبل ما تضغط **Approve** اتأكد من كل دول:

- [ ] **مفيش بيانات حقيقية:** مفيش IPs، hostnames، domains، usernames، passwords، API keys، enrollment tokens، أو certificates. لازم placeholders زي `<host-01>`, `<DC01>`, `<user>`, `10.0.0.x`, `example.local`, `<ELASTIC_PASSWORD>`.
- [ ] **الفولدر واسم الملف صح:** الملف في الفولدر المناسب (`01-architecture/` … `06-soar-ai/`, `final-report/`)، والاسم lowercase kebab-case، والـ rules `DR-XXX-short-name.md` والـ triage reports `TR-XXX-short-name.md`.
- [ ] **`Closes #N` موجود** في الـ description ورقم الـ issue صح.
- [ ] **الـ screenshots متنضفة:** IPs وأسامي الأجهزة والـ users والـ tokens متغطية أو متقصوصة، واسم الملف `YYYYMMDD_member_task.png`.
- [ ] مفيش raw logs أو captures (`*.evtx`, `*.pcap`).
- [ ] الـ IOCs في التقارير defanged (`hxxp://`, `1.2.3[.]4`).

---

## 7. تحريك الكروت على الـ SOC Project Board

كل issue ليه كارت على **SOC Project Board** (من الـ org `depi-soc-team` ← تاب **Projects**).

| الـ Status | إمتى |
|---|---|
| **Todo** | لسه مابدأتش |
| **In Progress** | أول ما تعمل الـ branch وتبدأ شغل |
| **Review** | أول ما تفتح الـ PR |
| **Done** | بعد الـ merge |

**إزاي تحرّك الكارت:**

- **من الـ board:** اسحب الكارت (drag) للعمود الصح.
- **من صفحة الـ issue:** في الـ sidebar على اليمين ← **Projects** ← **Status** ← اختار.

> لما الـ PR يتعمله merge والـ issue يتقفل بـ `Closes #N`، الكارت غالباً بيروح **Done** لوحده. لو مارحش، حرّكه بإيدك.

---

## 8. مشاكل شائعة وحلها

### `fatal: not a git repository`

انت مش جوه فولدر الـ repo.

```powershell
cd C:\dev\SIEM-Detection-Alert-Triage-SOC-Ops
git status
```

### الـ push اترفض (`rejected` / `fetch first` / `non-fast-forward`)

في حد عمل push على نفس الـ branch قبلك. اعمل pull الأول وبعدين push:

```powershell
git pull
git push
```

في GitHub Desktop: **Fetch origin** ← **Pull origin** ← **Push origin**.

### Merge conflict

بيحصل لما اتنين يعدّلوا نفس السطور. Git بيحط علامات في الملف:

```text
<<<<<<< HEAD
النسخة بتاعتك
=======
النسخة التانية
>>>>>>> main
```

1. افتح الملف (GitHub Desktop بيقولك على الملفات اللي فيها conflict ← **Open in Visual Studio Code**).
2. اختار النسخة الصح (أو ادمجهم)، و**امسح العلامات** `<<<<<<<` و `=======` و `>>>>>>>`.
3. احفظ الملف وبعدين:

```powershell
git add <file>
git commit
git push
```

في GitHub Desktop: بعد ما تصلح كل الملفات اضغط **Continue merge**.

> لو مش فاهم الـ conflict، ماتخمّنش — اسأل صاحب التعديل التاني أو الـ Lead.

### عملت commit على `main` بالغلط

**لو لسه ماعملتش push:** انقل الـ commit لـ branch جديد ورجّع `main` زي ما كان:

```powershell
git branch <member>/my-task
git reset --hard origin/main
git switch <member>/my-task
```

1. السطر الأول بيعمل branch جديد فيه الـ commit بتاعك.
2. السطر التاني بيرجّع `main` زي GitHub. ⚠️ اعمله **بعد** السطر الأول بس، وإلا هتخسر الـ commit.
3. السطر التالت بينقلك على الـ branch الجديد — كمّل عادي (push ← PR).

في GitHub Desktop: تاب **History** ← كليك يمين على الـ commit ← **Undo commit**. التغييرات هترجع **Changes**، وبعدين **Current branch > New branch** واختار **Bring my changes to <member>/my-task**.

**لو عملت push على `main`:** ماتحاولش تصلحها لوحدك — كلم الـ Lead.

### عملت commit لـ secret بالغلط (password, API key, token, IP حقيقي)

1. **بلّغ الـ Lead فوراً** — حتى لو لسه ماعملتش push.
2. **ماتكتفيش إنك تمسحه في commit جديد.** الـ secret هيفضل موجود في الـ Git history وأي حد يقدر يشوفه.
3. الـ secret ده يعتبر **اتسرب**: لازم يتغير (password جديد، token جديد) في المكان الأصلي.
4. الـ Lead هيتولى تنضيف الـ history.

---

## 9. Cheat sheet

| عايز تعمل إيه | Command line | GitHub Desktop |
|---|---|---|
| تشوف حالة الملفات | `git status` | تاب **Changes** |
| تحدّث `main` | `git switch main` ثم `git pull` | **Current branch** = `main` ← **Fetch origin** ← **Pull origin** |
| تعمل branch جديد | `git switch -c <member>/<short-task>` | **Current branch > New branch** |
| تتنقل لـ branch موجود | `git switch <branch>` | **Current branch** ← اختار |
| تضيف ملف للـ commit | `git add <file>` | علّم ✔ جنب الملف في **Changes** |
| تراجع قبل الـ commit | `git diff --staged` | اضغط على الملف في **Changes** |
| تعمل commit | `git commit -m "message"` | **Summary** ← **Commit to ...** |
| أول push للـ branch | `git push -u origin <branch>` | **Publish branch** |
| push بعد كده | `git push` | **Push origin** |
| تجيب آخر تعديلات | `git pull` | **Fetch origin** ← **Pull origin** |
| تشوف الـ history | `git log --oneline` | تاب **History** |
| تمسح branch بعد الـ merge | `git branch -d <branch>` | **Branch > Delete...** |
| تفتح PR | افتح اللينك بعد الـ push | **Preview Pull Request** ← **Create pull request** |

**افتكر دايماً:** pull main ← branch لكل task ← commit صغير وواضح ← PR فيه `Closes #N` ← review ← merge ← delete branch.
