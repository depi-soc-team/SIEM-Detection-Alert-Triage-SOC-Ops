# الوصول للـ Lab عن بُعد (Tailscale)

الدليل ده بيشرح إزاي توصل للـ lab والـ Kibana من بيتك عن طريق **Tailscale**، خطوة بخطوة.

> 🔒 **كل القيم في الدليل ده placeholders.** الـ IPs الحقيقية واسم الـ tailnet وأسامي الأجهزة ولينكات الدعوة **مش هتتكتب في الـ repo أبداً** — الـ Lead بيبعتها لكل واحد بشكل خاص.

| Placeholder | معناه |
|---|---|
| `<KIBANA_TS_ADDRESS>` | الـ Tailscale IP بتاع الـ SIEM VM (بيبدأ بـ `100.`) |
| `<SIEM_HOSTNAME>` | اسم الـ SIEM VM في Tailscale |
| `<TAILNET>` | اسم الـ tailnet (الجزء اللي قبل `.ts.net`) |
| `<LAB_SUBNET>` | الـ subnet بتاع الـ lab، مثلاً `10.0.0.0/24` |
| `<LAB_IP>` | IP جهاز جوه الـ lab، مثلاً `10.0.0.x` |

## المحتويات

1. [ليه Tailscale؟](#1-ليه-tailscale)
2. [طريقتين للوصول](#2-طريقتين-للوصول)
3. [خطوات العضو (Windows)](#3-خطوات-العضو-windows)
4. [خطوات الـ Admin](#4-خطوات-الـ-admin)
5. [مشاكل شائعة وحلها](#5-مشاكل-شائعة-وحلها)
6. [ملاحظات أمنية](#6-ملاحظات-أمنية)

---

## 1. ليه Tailscale؟

- الـ lab كله **VMs على جهاز host واحد** متوصل بإنترنت بيت عادي.
- الإنترنت ده ورا **CGNAT** — يعني مفيش **public IP**، ومينفعش حد من برة يوصل للجهاز مباشرة حتى لو عملنا port forwarding.
- الـ **FortiGate VM** شغال بـ **permanent trial license**، وده بيدعم **low encryption** بس — فالـ **SSL-VPN** بتاعه مش ينفع نستخدمه.
- **Tailscale** بيعمل اتصال **encrypted** مبني على **WireGuard** بين الأجهزة مباشرة، **من غير ما نفتح أي port** على الراوتر، وبيشتغل عادي ورا CGNAT.

> تفاصيل القرار والبدائل اللي اتقارنت: [ADR-001](../01-architecture/adr/ADR-001-remote-access-tailscale.md).

---

## 2. طريقتين للوصول

| | (أ) Node sharing — **الـ default** | (ب) Tailnet user + subnet router |
|---|---|---|
| بتوصل لإيه | الـ SIEM VM بس (Kibana) | الـ lab كله (`<LAB_SUBNET>`) |
| لمين | أغلب التيم | الأدوار اللي محتاجة الـ lab كله بس (زي **Infra**) |
| الـ Admin بيعمل إيه | **Machines** ← الـ SIEM VM ← **...** ← **Share...** | **Users** ← **Invite users** |
| بتوصل إزاي | `http://<KIBANA_TS_ADDRESS>:5601` | الـ Tailscale IP **أو** الـ lab IP (`<LAB_IP>`) |
| Least privilege | ✅ | ⚠️ صلاحية أوسع — بس للي محتاجها |

**(أ) Node sharing:** الـ Admin بيعمل share للـ SIEM VM بس. الجهاز بيظهر عندك في الـ Tailscale بتاعك، بس **مش بتشوف أي حاجة تانية في الـ lab**. ده الـ default لأنه **least privilege**.

**(ب) Tailnet user + subnet router:** بتبقى عضو في الـ tailnet نفسه، وبتوصل للـ lab subnet كله عن طريق الـ **subnet router**. ده بس للي شغله محتاج يوصل لأجهزة الـ lab مباشرة (FortiGate، الـ VMs، SSH...).

> لو مش عارف انت أنهي طريقة — يبقى (أ). اسأل الـ Lead لو محتاج (ب).

---

## 3. خطوات العضو (Windows)

### 3.1 سطّب Tailscale

1. نزّل Tailscale for Windows من <https://tailscale.com/download> وسطّبه.
2. هتلاقي أيقونة Tailscale في الـ system tray (جنب الساعة).

### 3.2 اقبل الدعوة

1. افتح لينك الـ **share** أو الـ **invite** اللي الـ Lead بعتهولك بشكل خاص.
2. اعمل **Sign in with GitHub** — استخدم نفس أكونت GitHub بتاع الـ repo.
3. اقبل الدعوة (**Accept**).
4. من أيقونة Tailscale في الـ tray اتأكد إنك **Connected**.

### 3.3 اتأكد إن الاتصال شغال

افتح PowerShell:

```powershell
tailscale status
```

لازم تلاقي الـ SIEM VM في الليستة (باسم `<SIEM_HOSTNAME>`).

```powershell
tailscale ping <KIBANA_TS_ADDRESS>
```

لو طلع `pong from ...` يبقى تمام.

### 3.4 افتح Kibana

في المتصفح افتح واحد من دول:

```text
http://<KIBANA_TS_ADDRESS>:5601
http://<SIEM_HOSTNAME>.<TAILNET>.ts.net:5601
```

> ⚠️ اكتب **`http://`** مش `https://`.

ادخل بالـ **Kibana account الشخصي بتاعك** (مش `elastic`) — شوف [02-onboarding/kibana-rbac.md](../02-onboarding/kibana-rbac.md).

**لو انت (ب) — tailnet user:** تقدر كمان توصل لأجهزة الـ lab بالـ lab IP مباشرة (`<LAB_IP>`). على Windows الـ subnet routes بتتقبل تلقائي؛ لو مش شغالة، من أيقونة الـ tray اتأكد إن **Use Tailscale subnets** متعلّم عليها.

---

## 4. خطوات الـ Admin

### 4.1 Node sharing (الطريقة أ)

1. افتح الـ **Tailscale admin console** ← **Machines**.
2. جنب الـ SIEM VM اضغط **...** ← **Share...**
3. ابعت اللينك أو الدعوة للعضو **بشكل خاص** (مش في الـ repo ولا في جروب عام).

### 4.2 Tailnet user (الطريقة ب)

1. **Users** ← **Invite users**.
2. ابعت الدعوة للعضو بشكل خاص.

### 4.3 Subnet router على Ubuntu

على الـ VM اللي هتبقى **subnet router** (Ubuntu):

1. سطّب Tailscale:

```bash
curl -fsSL https://tailscale.com/install.sh | sh
```

2. شغّله واعمل login:

```bash
sudo tailscale up
```

3. فعّل **IP forwarding** (لازم عشان الجهاز يعدّي traffic للـ lab):

```bash
echo 'net.ipv4.ip_forward = 1' | sudo tee /etc/sysctl.d/99-tailscale.conf
echo 'net.ipv6.conf.all.forwarding = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
```

4. أعلن عن الـ lab subnet:

```bash
sudo tailscale set --advertise-routes=<LAB_SUBNET>
```

5. **Approve the route:** في الـ admin console ← **Machines** ← الـ subnet router ← **...** ← **Edit route settings** ← علّم على `<LAB_SUBNET>` ← **Save**.
6. **Disable key expiry:** ← **Machines** ← **...** ← **Disable key expiry** للـ subnet router والـ SIEM VM — عشان مايفصلوش لوحدهم بعد فترة.

### 4.4 Kibana لازم يسمع على الـ Tailscale interface

لو Tailscale متسطب على الـ SIEM VM نفسها، Kibana لازم يقبل connections على الـ Tailscale IP كمان. في `kibana.yml`:

```yaml
# مايتربطش بـ lab IP واحد بس — وإلا الـ Tailscale IP مش هيشتغل
server.host: "0.0.0.0"
```

بعدها:

```bash
sudo systemctl restart kibana
```

> لو في firewall على الـ VM (زي `ufw`)، اسمح بـ port `5601` على الـ interface `tailscale0`.

---

## 5. مشاكل شائعة وحلها

| المشكلة | السبب | الحل |
|---|---|---|
| "This site can't be reached" على الـ **lab IP** بس شغال على الـ **Tailscale IP** | انت متعملك **node share** مش **invite** — الـ subnet routes **مابتتشيرش** مع الـ shared nodes | ده طبيعي. استخدم `http://<KIBANA_TS_ADDRESS>:5601`. لو دورك محتاج الـ lab كله، اطلب من الـ Lead invite (طريقة ب) |
| الـ SIEM VM مش ظاهرة في `tailscale status` | الدعوة ماتقبلتش، أو انت عامل sign in بأكونت تاني، أو الـ VM نفسها offline | افتح لينك الدعوة تاني واقبله بنفس أكونت GitHub. اتأكد إن Tailscale **Connected**. لو لسه، كلم الـ Lead يتأكد إن الـ VM شغالة |
| الـ lab IPs مش شغالة حتى وانت tailnet user | الـ subnet route لسه **مش approved** في الـ admin console | الـ Admin يعمل approve للـ route (خطوة 4.3 رقم 5) |
| الـ lab IPs بتروح لمكان غلط أو مش بترد | **VMware subnet conflict:** الـ VMware على جهازك (VMnet1/VMnet8) عامل نفس الـ subnet بتاع الـ lab | في VMware ← **Edit > Virtual Network Editor** غيّر subnet الـ VMnet، أو اقفل الـ adapter ده مؤقتاً |
| المتصفح بيقول connection error أو SSL error | كتبت `https://` أو المتصفح حوّل لـ https لوحده | اكتب `http://` صراحةً. Kibana في الـ lab شغال **HTTP** |

---

## 6. ملاحظات أمنية

- **أكونت شخصي لكل واحد** في Tailscale وفي Kibana. مفيش أكونتات مشتركة.
- **ماتشاركش الـ `elastic` superuser أبداً** — ده للـ Admin بس وقت الإعداد. كل واحد يدخل بأكونته وبالـ role المناسب ليه.
- **Kibana شغال HTTP جوه الـ tunnel:** الـ traffic encrypted بـ WireGuard من جهازك للـ VM، بس Kibana نفسه مش HTTPS. ده **known lab limitation** ومقبول في الـ lab بس — مش في production.
- **سحب الصلاحية:** لما حد يخلص أو يسيب التيم، الـ Admin بيشيله من **Users** (طريقة ب) أو بيلغي الـ **share** من الـ SIEM VM (طريقة أ)، وبيعمل disable لأكونته في Kibana.
- **ماتكتبش أي قيمة حقيقية** (IP، اسم tailnet، hostname، لينك دعوة) في الـ repo أو في screenshot من غير ما تغطيها.
