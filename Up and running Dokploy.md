# راه‌اندازی Dokploy

### مرحله 1. آپدیت سیستم و نصب package‌ ها

اول از همه، وارد سرورت شو و لیست پکیج‌ها رو که توی فایل `/etc/apt/sources.list` و دایرکتوری `/etc/apt/sources.list.d` هست، آپدیت کن.

```bash
sudo apt update
```

چند تا package باید نصب بشه تا مشکلی پیش نیاد. پس کد زیر رو اجرا کن.

```bash
sudo apt install curl apt-transport-https ca-certificates software-properties-common
```

### مرحله 2. نصب Docker

دو روش برای نصب Docker روی Ubuntu هست. اول، می‌تونی از مخازن پیش‌فرض با استفاده از APT نصبش کنی.

```bash
sudo apt install docker.io -y
```

ولی این نسخه آخرین نسخه نیست. برای نصب جدیدترین نسخه، باید از مخزن رسمی Docker نصبش کنی.

برای این کار، کلید GPG Docker رو با استفاده از کد `curl` دانلود کن.

```bash
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

بعد، مخزن APT Docker رو به سیستم اضافه کن. این کد یه فایل مخزن `docker.list` توی دایرکتوری `/etc/apt/sources.list.d` ایجاد می‌کنه.

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

حالا لیست پکیج‌های محلی رو آپدیت کن تا سیستم از مخزن جدید مطلع بشه.

```bash
sudo apt update
```

الان Docker Community Edition (رایگان برای دانلود و استفاده) رو به‌صورت زیر نصب کن. توی این کد، گزینه `-y` اجازه نصب غیرتعاملی رو می‌ده.

```bash
sudo apt install docker-ce -y
```

سرویس Docker به‌طور خودکار بعد از نصب شروع می‌شه. می‌تونی وضعیتش رو با اجرای کد زیر چک کنی:

```bash
sudo systemctl status docker
```

### مرحله 3. اضافه کردن کاربر به گروه Docker

به‌طور پیش‌فرض، Docker به‌صورت root یا کاربر عادی با امتیازات بالا (کاربر sudo) اجرا می‌شه. این یعنی فقط کاربر root یا sudo می‌تونه کدهای Docker رو اجرا کنه. اجرای کدهای docker به‌عنوان کاربر عادی منجر به پیام `'permissions error'` می‌شه.

به طور مثال ما یک کاربر به نام `cherry` داریم و چون ما قبلاً یه کاربر `sudo` به نام `cherry` پیکربندی نکردیم، باید کاربر فعلی وارد شده رو به گروه `docker` اضافه کنیم. این اطمینان می‌ده که نیازی به فراخوانی `sudo` هنگام اجرای کدهای Docker نداریم چون کاربر قبلاً به گروه `docker` تعلق داره.

برای اضافه کردن کاربر فعلی وارد شده، از کد usermod استفاده کن.

```bash
sudo usermod -aG docker $USER
```

بعد، کد `newgrp` رو اجرا کن.

```bash
newgrp
```

حالا کد groups رو اجرا کن تا تأیید کنی که کاربر به گروه docker اضافه شده.

```bash
groups cherry
```

برای شروع اجرای کدهای Docker بدون فراخوانی `sudo`، یه session جدید باز کن یا کد زیر رو بدون بستن session فعلی خودت اجرا کن.

```bash
su -$USER
```

از این به بعد، می‌تونی کدهای Docker رو به‌عنوان کاربر عادی به‌طور روان اجرا کنی. مثلاً می‌تونی نسخه Docker رو با اجرای کد زیر چک کنی:

```bash
docker version
```

### مرحله 4. نصب داکر و حل مشکلاتش روی سرور ایران

یکی از میرورهای داکر رو داخل فایل `daemon.json` ذخیره می‌کنیم.

```bash
nano /etc/docker/daemon.json
```

#### IranServer

```json
{
	"registry-mirrors": ["https://docker.iranserver.com"]
}
```

#### HaioCloud

```json
{
	"registry-mirrors": ["https://docker.haiocloud.com"]
}
```

#### ParsPack

```json
{
	"registry-mirrors": ["https://registry.docker.ir"]
}
```

#### AbrArvan

```json
{
	"registry-mirrors": ["https://docker.arvancloud.ir"]
}
```

#### Dockeriran

```json
{
	"registry-mirrors": ["https://docker.host:5000"]
}
```

بعد سرویس داکر رو ریستارت کن.

```bash
systemctl daemon-reload
systemctl restart docker
```

[توضیحات بیشتر](https://github.com/Gozargah/Marzban/discussions/987)

### مرحله 5. تست نصب Docker

تا این مرحله، Docker با موفقیت نصب شده. قبل از ادامه، باید مطمئن بشی که می‌تونی image ها رو از Docker Hub، مخزن پیش‌فرض Docker که برای اجرا با Docker پیکربندی شده، بگیری و container ها رو اجرا کنی. برای تست این، ما یه container از image `hello-world` ایجاد می‌کنیم.

```bash
docker run hello-world
```

#### دقیقاً چه اتفاقی افتاد؟

در پس‌زمینه، کلاینت Docker به دنبال image ای به نام `hello-world` با تگ `latest` توی سیستم محلی گشت ولی پیداش نکرد. بعد با Docker Hub تماس گرفت، image رو به سیستم محلی کشید و یه container جدید از اون image ایجاد کرد. container بعد خروجی رو روی ترمینال شما استریم کرد و خارج شد.

### مرحله 6. نصب Dokploy

#### نیازمندی‌ها

برای اطمینان از تجربه‌ای روان با Dokploy، سرور شما باید حداقل `2GB` **RAM** و `30GB` **فضای دیسک** داشته باشه. این مشخصات به مدیریت منابع مصرفی توسط Docker در طول ساخت‌ها کمک می‌کنه و از فریز شدن سیستم جلوگیری می‌کنه.

Dokploy از Docker استفاده می‌کنه، پس لازمه Docker روی سرور شما نصب شده باشه.

```bash
curl -sSL https://dokploy.com/install.sh | sh
```
