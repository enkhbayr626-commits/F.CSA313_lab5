# Lab05 — API систем тест Postman ба Newman

**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест (2026)

**Оюутан:** С. Энхбаяр

**Оюутны код:** B232270153

## Орчин ба анхны амжилттай тест

API сервер болон Newman-ийг WSL2 Ubuntu дотор ажиллуулна. Багшийн өгсөн API кодыг өөрчлөлтгүйгээр `server.js` нэрээр хадгалсан. Сервер нэмэлт npm сан шаарддаггүй, өгөгдлийг санах ойд хадгална.

Бодитоор шалгасан орчны хувилбарууд:

```text
$ node -v
v18.19.1
$ newman -v
6.2.3
$ git --version
git version 2.43.0
```

Newman-ийг хэрэглэгчийн эрхээр суулгах:

```bash
npm install --prefix ~/.local --global newman@6
export PATH="$HOME/.local/bin:$PATH"
newman -v
```

`export PATH` командыг шинэ терминал бүрд ажиллуулна, эсвэл `~/.bashrc` файлдаа нэмнэ.

## Ажиллуулах

WSL Ubuntu-ийн эхний терминалд:

```bash
cd ~/projects/F.CSA313_lab5
node server.js
```

Серверийн хаяг: `http://localhost:3000`. Өөр WSL терминалд:

```bash
cd ~/projects/F.CSA313_lab5
export PATH="$HOME/.local/bin:$PATH"
newman run lab05-collection.json --color off
echo "exit=$?"
```

Collection нь Postman Collection v2.1 JSON форматтай. Postman-д Import → файл сонгож оруулна. Collection variable `baseUrl` нь `http://localhost:3000`; JSON body-той хүсэлтүүд `Content-Type: application/json` header ашиглана.

## Анхны кейс ба oracle

`TC01 — Урьдач нөхцөл хангасан идэвхтэй оюутан` folder дарааллаар гурван хүсэлт ажиллуулна:

1. `PUT /students/B232270153`: `status=active`, `coursesTaken=["CS201"]`.
2. `PUT /courses/CS313`: `prerequisites=["CS201"]`.
3. `POST /registrations`: `studentID=B232270153`, `courseID=CS313`.

POST хүсэлтэд тусдаа хоёр `pm.test` oracle байна: HTTP статус `201`, хариуны `result` утга `OK`. `registrationID`-ийн тогтмол утгыг шалгахгүй. Кейс өөрийн setup-аа гүйцэтгэдэг тул дахин ажиллуулах боломжтой.

## Шалгалтын үр дүн

WSL Ubuntu дээр ижил серверийг дахин асаахгүйгээр collection-ийг хоёр удаа ажиллуулсан. Хоёр ажиллуулалт хоёулаа дараах үр дүнтэй:

| Үзүүлэлт | Executed | Failed |
|---|---:|---:|
| Iterations | 1 | 0 |
| Requests | 3 | 0 |
| Assertions | 2 | 0 |

Хоёр ажиллуулалтын exit code: `0`. Эхний ажиллуулалтын бүтэн гаралт: [Newman-ийн гаралт](results/stage1-newman.txt).
