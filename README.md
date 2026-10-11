# Lab05 — API систем тест Postman ба Newman

**Хичээл:** F.CSA313 — Программ хангамжийн чанарын баталгаа ба тест (2026)

**Оюутан:** С. Энхбаяр — **B232270153**

## Орчин ба ажиллуулах

API болон Newman-ийг WSL2 Ubuntu дээр ажиллуулсан. Багшийн `server.js` кодыг өөрчлөөгүй; нэмэлт сан шаардахгүй, өгөгдөл нь санах ойд хадгалагдана.

```text
$ node -v
v18.19.1
$ newman -v
6.2.3
```

Newman суулгах:

```bash
npm install --prefix ~/.local --global newman@6
export PATH="$HOME/.local/bin:$PATH"
```

Эхний WSL терминалд `cd ~/projects/F.CSA313_lab5` → `node server.js` ажиллуулна. Өөр терминалд:

```bash
cd ~/projects/F.CSA313_lab5
export PATH="$HOME/.local/bin:$PATH"
newman run lab05-collection.json --color off
echo "exit=$?"
```

Collection нь Postman v2.1 JSON форматтай, Postman-д Import хийж ашиглана. Collection variable `baseUrl=http://localhost:3000`; JSON хүсэлтүүд `Content-Type: application/json` ашиглана.

## Тест дизайн

Лекцийн таван алхмыг хэрэглэсэн: **тестлэгдэх функц → сонголтууд → төлөөлөх утгууд → спецификаци → бодит кейс**. Тестлэх функц нь `POST /registrations`; `PUT /students/:id`, `PUT /courses/:id` нь setup хүсэлтүүд юм.

### Сонголт ба төлөөлөх утгууд

| Сонголт | Эквивалент анги | Төлөөлөх утга |
|---|---|---|
| Оюутны хүчинтэй байдал | Байгаа, идэвхтэй | `status="active"` |
| Оюутны хүчинтэй байдал | Байгаа, идэвхгүй | `status="inactive"` |
| Оюутны хүчинтэй байдал | Байхгүй | PUT-ээр үүсгээгүй ID |
| Үзсэн хичээлүүд | Урьдач нөхцөл хангана | `coursesTaken=["CS201","CS202"]`, `prerequisites=["CS201","CS202"]` |
| Үзсэн хичээлүүд | Урьдач нөхцөл хангахгүй | `coursesTaken=[]` эсвэл `["CS201"]`, `prerequisites=["CS201","CS202"]` |
| Хичээлийн хүчинтэй байдал | Байгаа | PUT-ээр үүсгэсэн ID |
| Хичээлийн хүчинтэй байдал | Байхгүй | PUT-ээр үүсгээгүй ID |
| Урьдач нөхцөл | Бүгдийг үзсэн | `prerequisites=["CS201"]`, `coursesTaken=["CS201"]` |
| Урьдач нөхцөл | Огт үзээгүй | `prerequisites=["CS201","CS202"]`, `coursesTaken=[]` |
| Урьдач нөхцөл | Заримыг үзсэн | `prerequisites=["CS201","CS202"]`, `coursesTaken=["CS201"]` |
| Урьдач нөхцөл | Нөхцөлгүй | `prerequisites=[]`, `coursesTaken=[]` |
| Хүсэлтийн талбарууд | Бүрэн / дутуу | Хоёр ID-тай объект / аль нэг ID-г орхисон объект |
| JSON формат | Зөв / буруу | JSON объект / raw `{"studentID":` |

### Спецификаци

TC01 нь `B232270153`, `CS313`; бусад кейс `B232270153_TCnn`, `CS313_TCnn` гэсэн тусдаа ID ашиглана. Кейс бүр шаардлагатай бичлэгээ өөрөө үүсгэж, байхгүй гэж шалгах ID-г бусад кейс үүсгэхгүй.

| Кейс | Нөхцөл | Статус | `result` | Нэмэлт хүлээлт |
|---|---|---:|---|---|
| TC01 | Идэвхтэй, үзсэн болон урьдач хичээл `["CS201"]` | 201 | `OK` | `registrationID` тоо |
| TC02 | Оюутан байхгүй, хичээл байгаа | 200 | `ERROR_NO_STUDENT` | — |
| TC03 | Оюутан `inactive`, хичээл байгаа | 200 | `ERROR_INACTIVE_STUDENT` | — |
| TC04 | Идэвхтэй оюутан, хичээл байхгүй | 200 | `ERROR_NO_COURSE` | — |
| TC05 | Идэвхтэй, `coursesTaken=[]`, урьдач `["CS201","CS202"]` | 200 | `ERROR_PREREQUISITES` | `missing=["CS201","CS202"]` |
| TC06 | Идэвхтэй, үзсэн `["CS201"]`, урьдач `["CS201","CS202"]` | 200 | `ERROR_PREREQUISITES` | `missing=["CS202"]` |
| TC07 | Оюутан ба хичээл хоёулаа байхгүй | 200 | `ERROR_NO_STUDENT` | Оюутны алдаа түрүүлнэ |
| TC08 | Оюутан `inactive`, хичээл байхгүй | 200 | `ERROR_INACTIVE_STUDENT` | Төлөвийн алдаа түрүүлнэ |
| TC09 | Идэвхтэй, `coursesTaken=[]`, `prerequisites=[]` | 201 | `OK` | `registrationID` тоо |
| TC10 | `studentID` талбар дутуу | 400 | `ERROR_BAD_REQUEST` | Зөвхөн `courseID` илгээнэ |
| TC11 | `courseID` талбар дутуу | 400 | `ERROR_BAD_REQUEST` | Зөвхөн `studentID` илгээнэ |
| TC12 | Буруу JSON | 400 | `ERROR_BAD_JSON` | Raw `{"studentID":` |

TC03, TC04, TC08, TC10–TC12-ийн байгаа бичлэгүүдэд үзсэн болон урьдач хичээлийг `["CS201"]` гэж бэлдсэн; TC02-д зөвхөн хичээл, TC07-д ямар ч бичлэг үүсгэхгүй. Бүх POST-д статус, `result`-ийг тусдаа oracle-оор, setup PUT-д `200`, `OK`-г шалгана. Амжилттай бүртгэлийн ID-г тогтмол утгатай харьцуулахгүй, тоо эсэхийг шалгана.

Давхар алдааны тестүүд **талбарууд → оюутан → төлөв → хичээл → урьдач нөхцөл** дарааллыг баталсан. Байхгүй оюутны төлөвийг үнэлэх, байхгүй хичээлийн урьдач нөхцөлийг шалгах, урьдач нөхцөлгүй хичээлд нөхцөл дутуу гэж үзэх нь утгагүй хослолууд юм.

## Нийтийн API ба бие даасан байдал

`Public API` folder нь `GET https://jsonplaceholder.typicode.com/users` хүсэлттэй. Oracle-ууд: статус `200`, хариу массив, эхний хэрэглэгчийн нэр `Leanne Graham`. GET-only тул setup шаардлагагүй боловч интернет болон гаднын үйлчилгээний төлөвөөс хамаарна.

Бүтэн collection-ийг нэг сервер дээр хоёр удаа, 13 folder бүрийг шинэ сервертэй дангаар ажиллуулахад бүгд PASS болсон. Нотолгоо: [эхний ажиллуулалт](results/collection-pass.txt), [давтан ажиллуулалт](results/collection-repeat.txt), [бие даасан шалгалт](results/collection-independent.txt).

## Newman нотолгоо

Нийт **12 локал кейс**, нийтийн API-ийн нэг хүсэлттэй. Бүтэн collection **32 хүсэлт, 69 assertion** ажиллуулдаг; кейсийн тоо assertion-ы тоотой адил биш.

FAIL хуулбар `lab05-collection-fail.json`-д зөвхөн TC01-ийн POST статусын oracle-ийг `201`-ээс `200` гэж хүлээдэг болгосон. Үндсэн collection зөв хэвээр байна.

| Ажиллуулалт | Iterations executed / failed | Requests executed / failed | Assertions executed / failed | Exit | Бүтэн гаралт |
|---|---|---|---|---:|---|
| PASS | 1 / 0 | 32 / 0 | 69 / 0 | 0 | [newman-pass.txt](results/newman-pass.txt) |
| FAIL | 1 / 0 | 32 / 0 | 69 / 1 | 1 | [newman-fail.txt](results/newman-fail.txt) |
| DOWN | 1 / 0 | 32 / 31 | 69 / 66 | 1 | [newman-down.txt](results/newman-down.txt) |

FAIL нь зөв хариуг буруу хүлээлттэй харьцуулсан oracle-ийн алдаа; DOWN нь сервертэй холбогдохгүй `ECONNREFUSED` интерфейсийн алдаа. DOWN үед нийтийн API-ийн гурван oracle PASS хэвээр байна.

Нотолгоо хадгалах Bash жишээ; pipeline-ийн дараа Newman-ийн exit code-ийг шууд авна:

```bash
mkdir -p results
newman run lab05-collection.json --color off 2>&1 | tee results/newman-pass.txt
run_exit=${PIPESTATUS[0]}
printf '\nexit=%s\n' "$run_exit" | tee -a results/newman-pass.txt
```

FAIL-д хуулбар collection болон `newman-fail.txt` ашиглана. DOWN-д серверийг `Ctrl+C`-ээр зогсоож, үндсэн collection-ийг `newman-down.txt` руу ажиллуулна. Exit `0` бол quality gate тэнцэнэ, `1` бол зогсоно; дахин ажиллуулбал тайлангийн тоог гаралттай тааруулна.

## Дүгнэлт

Таван алхмаас эквивалент анги, утгатай хослол сонгох нь хамгийн их бодол шаардсан.
Үзсэн хичээл болон урьдач нөхцөл хоорондоо хамааралтай тул хамтад нь үнэлсэн.
Байхгүй оюутны төлөв, урьдач нөхцөлгүй хичээлийн дутуу нөхцөл зэрэг боломжгүй хослолыг хассан.
Тусдаа ID болон setup хүсэлтүүд тестийн бие даасан байдлыг хангасан.
Давтан болон дангаар ажиллуулсан шалгалтууд бүгд амжилттай болсон.
Давхар алдааны кейсүүд серверийн шалгах дарааллыг баталсан.
Шалгасан спецификацийн хүрээнд API-ийн согог илрээгүй.
Зориудын FAIL нь буруу oracle алдаа заадгийг харуулсан.
DOWN нь холболтын алдааг oracle-ийн алдаанаас ялгах боломж олгосон.
Бүтэн гаралт, exit code нь үр дүнг нотолж, автомат quality gate-д ашиглагдана.
