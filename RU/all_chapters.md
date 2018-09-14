## Как создавать базирующиеся на UTXO (выходах) контракты CryptoConditions для блокчейнов в кластере KMD - автор jl777

Исходный текст: https://github.com/jl777/komodo/blob/FSM/src/cc/CC%20made%20easy

Это не единственная методология смарт-контрактов, которую возможно построить на OP_CHECKCRYPTOCONDITION, просто она является первой. Вся заслуга за то, что OP_CHECKCRYPTOCONDITION работает с кодовой базой Komodo принадлежит @libscott. Я просто цепляюсь к коду и пытаюсь сделать его немного проще для создания новых контрактов.

Пожалуй можно придумать какое нибудь причудливое имя в целях маркетинга, но сейчас я просто коротко назову это "CC контракты", зная что это не на 100% технически точно, так как аспект CryptoConditions не является главной характерной чертой. Однако, контракты KMD были построены на кодовой базе CryptoConditions, которая была интегрирована в них, чтобы стать более доступной.
Поскольку CC контракты запускают нативный C/C++ код, они являются Тьюринг-полными, а это значит, что, любой контракт который возможно создать на любой другой платформе можно реализовать через CC контракт.

Начать писать контракты на основе UTXO немного сложнее чем для контрактов основанных на баланах. Однако, они гораздо более безопасны, поскольку используют существующую систему UTXO Bitcoin. Это делает гораздо менее вероятным появление багов, из-за которых можно выпустить квадриллион новых монет, поскольку все операции с CC контрактом также должны подчиняться существующему UTXO протоколу Bitcoin.

Данный документ будет в значительной степени основан на примерах, то есть будет использовать много отсылок к существующим CC. После осознания данного документа, вы должны быть достаточно подготовлены чтобы начать создавать новые CC контракты для интеграции в komodod, либо непосредственно dAPPS на основе RPC.
- [Глава 0 - Основы протокола Bitcoin](#Глава-0---Основы-протокола-bitcoin)
- [Глава 1 - OP_CHECKCRYPTOCONDITION](#Глава-1---op_checkcryptocondition)
- [Глава 2 - Основы СС контрактов](#Глава-2---Основы-СС-контрактов)
- [Глава 3 - CC vins и vouts](#Глава-3---cc-vins-и-vouts)
- [Глава 4 - RPC расширения СС](#Глава-4---cc-rpc-extensions)
- [Глава 5 - Валидация СС](#Глава-5---Валидация-СС)
- [Глава 6 - faucet example](#chapter-6---faucet-example)
- Глава 7 - rewards example
- Глава 8 - assets example
- Глава 9 - dice example
- Глава 10 - lotto example
- Глава 11 - channels example
- [Глава 12 - limitless possibilities](#chapter-12---limitless-possibilities)
- [Глава 13 - different languages](#chapter-13---different-languages)
- [Глава 14 - runtime bindings](#chapter-14---runtime-bindings)
- [Глава 15 - RPC based dapps](#chapter-15---rpc-based-dapps)
- [Заключение](#conclusion)



## Глава 0 - Основы протокола Bitcoin
Существует множество аспектов протокола Bitcoin, для которых не требуется понимание зависимости от них СС контрактов. Подобные детали не будут обсуждаться. Главный аспект - `UTXO`, неизрасходованный выход транзакции. Это просто причудливое имя для txid/vout, то есть когда вы отправляете (sendtoaddress) некоторые монеты это создает `txid` (идентификатор транзакии) и первый выход - `vout.0` совмещая его и `txid/0` получается определенная `UTXO`.

Конечно, чтобы получить понимание даже на этом уровне объяснения вам нужно знать что такое `txid`, однако существует много справочных материалов по данному вопросу. Собственно это просто набор букв и цифр длиной в 64 символа который вы получаете когда отправляете средства.

Безоговорочным для `UTXO` является то, что, они предотвращают двойные расходы. Как только вы потратили `UTXO`, вы не можете потратить его снова. Это достаточно важная характеристика, может быть наши продвинутые читатели укажут на то, что перестроение блокчейна может позволить двойное списание - мы не будет запутывать проблему такими деталями. Важно то, что на данном блокчейне на блокхэше конкретной высоты вы можете знать была ли потрачена `txid/vout` или нет.

Так же существуют транзакции которые находятся в памяти, ожидая пока их "смайнят" - `mempool`. И для `utxo` возможно быть потраченным транзакцией (`tx`) в `mempool`. Однако, так как это еще не подтверждено, он все еще непотрачен на текущей высоте, даже если мы достаточно точно уверены в том, что он будет потрачен в следующем блоке.

Неплохой пример - подумать об очереди людей, выстроившихся в линию чтобы попасть на событие. Им нужно иметь валидный билет а так же встать в очередь. По прошествии какого-то времени на их билете ставится отметка и они попадают на событие.

В случае с `UTXO`, билет это тратящая транзакция, а событие это подтвержденный блокчейн. Очередь это `mempool`.



## Глава 1 - OP_CHECKCRYPTOCONDITION
В предыдущей главе были объяснены UTXO. Однако конкретный механизм, используемый для отправки платежа, не объяснялся. Вопреки тому, что, думает большинство людей, на блокчейне нет записей в духе "заплати сумму X на такой-то адрес". Вместо этого существует биткоин-скрипт (https://en.bitcoin.it/wiki/Script), который должен быть удовлетворен для того, чтобы средства могли быть потрачены.

Первоначально в нем была оплата на pubkey (публичный ключ) скрипт:

```
EN: <pubkey> <checksig>
RU: <публичный ключ> <проверка валидности подписи транзакции>
```

Пожалуй выше самый простой платежный скрипт который вы можете сделать. Подпись pubkey проверятся, и если она валидная вы можете это потратить.  Одна из проблем которую понял Сатоши это то, что с квантовыми компьютерами подобные платежные скрипты уязывимы! Тогда он сделал способ иметь холодный адрес, то есть такой адрес чей pubkey не известен. Он неизвестен по крайней мере  до того момента как будет потрачен, то есть подобная схема защищена от квантовых компьютеров только до первой траты. Данная линия рассуждений это то, почему мы имеем одноразовые адреса и смену адреса для каждой транзакции. Возможно в некотором роде это мышление слишком наперед, поскольку это делает вещи гораздо более запутанными и легче потерять все необходимые приватные ключи.

Однако это общепринято и текущий скрипт:

```
EN: <hash the pubkey> <pubkey> <verify hash matches> <checksig>
RU: <хэш публичного ключа> <публичный ключ> <подтверждение совпадение хэшей> <проверка валидности подписи транзакции>
```

При этом блокчейн имеет то, что образует пару с "платежом на адрес", а имеено то, что адрес на самом деле закодирован в варианте Base58 (prefix + pubkeyhash, https://ru.wikipedia.org/wiki/Base58). Эй, если бы это было не сложно - это было бы легко!

Чтобы потратить `p2pkh` (pay to pubkey hash, хэш платежа на публичный ключ) UTXO, вам нужно разгласить публичный ключ в дополнение к валидной подписи. После первой траты с адреса его безопасность деградирует до `p2pk` (pay to pubkey, платеж на публичный ключ) так как теперь ее публичный ключ известен. Конечным результатом является то, что каждый повторно используемый адрес занимает 25 дополнительных байт в блокчейне, и это причина по которой для адресов с ожидаемым повторным использованием я просто использую `p2pk` скрипт.

Изначально биткоин позволял любой тип скриптов кодов операций (opcodes) для прямого использования. Трудность была в том, что некоторые из них вызывали проблемы и Сатоши решил отключить их и разрешить только стандартные формы платежей. Так `p2pk` и `p2pkh` стали 99%+ транзакций Bitcoin. Однако, переход от наличия полностью скритового языка который может создавать бесчисленные платежные скрипты (и баги!), чтобы иметь всего 2 (скрипта)... ну, это было "краткосрочное" ограничение. Это продолжалось в течение нескольких лет, но в конечном итоге компромиссный `p2sh` скрипт был признан стандартом. Это платеж хэшу скрипта (script hash), то есть он может обладать стандартным форматом как нормальный `p2pkh`, но иметь бесконечно больше гибкости.

```
EN: <hash the script> <script> <verify hash matches>
RU: <хэш скрипта> <скрипт> <подтверждение совпадения хэша>
```

Подождите, что-то не так! Если бы все было бы действительно так, тогда любой кто нашел требуемый скрипт (называемый `redeemscript`, скрипт высвобождения платежа) мог бы просто потратить транзакцию. Я забыл сказать что `redeemscript` затем используется чтобы оопределить был ли потрачен платеж или нет. Таким образом вы можете иметь нормальный `p2pk` либо `p2pkh` `redeemscript` внутри `p2sh` скрипта.

Окей, я знаю что это реально запутанно. Давайте посмотрим на более понятный пример:

```
redeemscript <- pay to pubkey
```

`p2sh` становится хэшом высвобождающего скрипта + сравнения

Поэтому, чтобы потратить его, вам нужно разгласить reedemscript, который, в свою очередь, требует от вас разглашения pubkey. Смешайте это все вместе и `p2sh` механизм  не только подтверждает сравнением хэша то, что у вас есть корректный reeedemscript, но и то, что когда reedemsctipt запущен он удовлетворен.  В этом случае подпись публичного ключа валидна.

Если вы все еще следите за нитью рассуждений - есть хорошие новости! `OP_CHECKCRYPTOCONDITION` скрипты на самом деле в некотором смысле проще чем p2sh скрипты, поскольку не имеют дополнительного уровня скриптов внутри scripthash. @libscott реализовал добавление `OP_CHECKCRYPTOCONDITION` в набор функций (opcodes) Bitcoin и что он делает, так это убеждается в том, что CryptoConditions скрипт правильно подписан.

Что дает нам спецификацию CryptoConditions, монстр от проекта [IETF (Internet standards) draft](https://tools.ietf.org/html/draft-thomas-crypto-conditions-04) с сотнями страниц спецификации. Я уверен что вы будете рады узнать что вам не требуется знать о ней много! Просто знайте, что вы можете создавать всевозможные cryptoconditions а его двоичное представление может быть использовано в UTXO Bitcoin. Если стандартные CC контракты не обладают необходимой вам мощностью, всегда возможно их всегда можно расширить. Пока что большинству CC контрактов требуется мощщность только 1из1 CC скрипта, в котором 1 подпись сочетается с заданными ограничениями. CC каналов оплаты в режиме реального времени  - единственный из стандартных CC контрактов, который не вписывался в эту модель, для него нужен 1из2 CC скрипт.

Самое приятное это то, что все эти вещи на уровне opcodes вообще не требуются. Я просто хотел объяснить это тем, кто хочет знать все в деталях.



## Глава 2 - Основы СС контрактов
Каждый CC контракт имеет "eval" код, это просто произвольное число, связанное с конкретным СС конрактом. Детали конкретного CC конртакта полностью определяются логикой валидации, которая в конечном итоге реализует CC контракт.

Однако, в отличие от обычных биткоин-платежей, которые проверяются только информацией в транзакции, СС контракт имеет право делать почти все что угодно. Он имеет полный доступ к блочейну и даже mempool, хотя использование информации из mempool является в своей сути более рискованным, и должно быть реализовано тщательно или для исключений, а не для включений.

Тем не менее эта глава называется "Основы" СС контрактов, поэтому давайте игнорировать проблемы с mempool и рассмотрим только основы. Фундаментально не существует структуры для сериализованных скриптов `OP_CHECKCRYPTOCONDITION`, но если вы такой же как и я - вы не хотите читать и понимать 1000 страниц стандарта IETF. То, что мы действительно хотим сделать - это логичный способ создать новый контракт и иметь возможность эффективно его кодировать и отлаживать.

Это означает простое следование известному рабочему шаблону и изменять только те вещи, где существующие шаблоны недостаточны, то есть основные отличительные черты вашего СС контракта.

В файле  [~/komodo/src/cc/eval.h](https://github.com/jl777/komodo/tree/jl777/src/cc/eval.h) представлены все eval коды. На данный момент он выглядит так:

```C
#define FOREACH_EVAL(EVAL)             \
        EVAL(EVAL_IMPORTPAYOUT, 0xe1)  \
        EVAL(EVAL_IMPORTCOIN,   0xe2)  \
        EVAL(EVAL_ASSETS,   0xe3)  \
        EVAL(EVAL_FAUCET, 0xe4) \
        EVAL(EVAL_REWARDS, 0xe5) \
        EVAL(EVAL_DICE, 0xe6) \
        EVAL(EVAL_FSM, 0xe7) \
        EVAL(EVAL_AUCTION, 0xe8) \
        EVAL(EVAL_LOTTO, 0xe9) \
        EVAL(EVAL_MOFN, 0xea) \
        EVAL(EVAL_CHANNELS, 0xeb) \
        EVAL(EVAL_ORACLES, 0xec) \
        EVAL(EVAL_PRICES, 0xed) \
        EVAL(EVAL_PEGS, 0xee) \
        EVAL(EVAL_TRIGGERS, 0xef) \
        EVAL(EVAL_PAYMENTS, 0xf0) \
        EVAL(EVAL_GATEWAYS, 0xf1)
```

В конечном счете, мы вероятно закончим использованием всех 256 eval кодов, сейчас есть много места. Я представил что с похожим на мои монеты репозиторием мы можем закончить с гораздо большим чем 256 числом СС контрактов и вы просто выберете 256 которые вы хотите активировать для вашего блокчейна. Это означает, что любой конкретный блокчейн будет ограничен наличием "только" 256 контрактов. Поскольку до сих пор, похоже, так мало действительно полезных контрактов, этого предела, по-видимому, достаточно. Я говорил о том, что eval-код может быть любой длины, но текущие CC контракты предполагают размер в 1 байт.

Простейшим CC скриптом будет тот, который требует подписи от публичного ключа вместе с СС валидацией. Это эквивалентно платежу на pubkey bitcoin скрипт и это то, что использует большинство СС контрактов. Только каналы требуют большего и я объясню это в данной главе.
В итоге мы получаем СС скрипты в форме: (`evalcode`) + (`pubkey`) + (`другие вещи`), не беспокойтесь насчет других вещей, они обрабатываются автоматически при помощи некоторых удобных внутренних функций. Важно отметить, что каждый СС контракт этой формы требует единчного публичного ключа и eval кода и из этого мы получаем CC скрипт. Используя стандартный метод bitcoin "берем хэш и делаем из него адрес", это значит что тот же публичный ключ будет генерировать разные адреса для каждого разного CC контракта!

Это важный момент, поэтому я расскажу об этом еще, но по-другому. В Bitcoin раньше использовались несжатые публичные ключи, имевшие в сумме при сложении правой и левой частей, огромный 64 байтовый публичный ключ. Но так как вы можете извлечь одно из другого, сжатые публичные ключи стали стандартом, вот почему ваши публичные ключи Bitcoin имеют размер 33 байта а не 65. Существуют префиксы 02, 03 или 04, чтобы обозначить нечетный, четный или большой публичный ключ. Это означает, что существует два разных публичных ключа для каждого приватного ключа, сжатый и несжатый. И действительно у вас могут быть два различных адреса протокола bitcoin которые могут быть потрачены одинаковым приватным ключом. Если вы используете генераторы бумажных кошельков - вы уже наверное это заметили.

СС контракты устроены так, что каждый публичный ключ получает другой адрес для каждого eval кода. Это тот же публичный ключ, разные адреса просто потому что фактический скрипт имеет другой eval код, он заканчивается на другой хэш и отсюда разные адреса. Теперь средства отправляемые на конкретный СС адрес доступны только через этот СС контракт и должны следовать правилам данного контракта.
Я так же добавил еще одну полезную функцию, в которой есть соглашение иметь специальный адрес для каждого СС контракта, известный всем, включая его приватный ключ. До того, как вы начнете паниковать по поводу публикации приватного ключа, запомните, что чтобы потратить выход СС, вы должны правильно подписать его И выполнить все правила. Все у кого есть приватный ключ для СС контракта могут сделать часть "правильной подписи", однако им все рвно нужно следовать оставшимся правилам.

С точки зрения пользователя, существует глобальный СС адрес для СС контракта и некоторые контракты так же используют СС адрес публичного ключа пользователя. Иметь пару новых адресов для каждого контракта может показаься немного запутанным на первый взгляд, но в итоге мы получим простой в использовании графический интерфейс.


## Глава 3 - CC vins и vouts
Возможно вам захочется ознакомиться с основами Bitcoin и другими материалами, чтобы освежить в голове то, как Bitcoin выходы становятся входами. Это немного сложно, но в конечном счете речь идет об одном конкретном количестве монет которые были потрачены, после траты это соединяется с другими, так же потраченными в данной транзакции, монетами, а затем создаются различные выходы.

```
vin0 + vin1 + vin2 -> vout0 + vout1
```

Это транзакция с 3-мя входами и 2-мя выходами. Сумма монет из трех выходов объединяется и затем делится на vout0 и vout1, каждый из vout-ов получает скрипт траты, который должен быть удовлетворен, для того чтобы быть потраченым. Это означает что все требования (так, как указаны в выходе которых их создал) удовлетворены для всех трех vin-ов.

Да, я знаю это может быть немного сложно без хорошей диаграммы, поэтому мы будем надеяться, что хороший график появится здесь:

`[хороший график будет здесь]`

Из всех аспектов CC контрактов, гибкость которую создали различные vins и vouts была самым большим сюрпризом. Когда я начинал писать первый из них месяц назад, я и понятия не имел о силе скрывающейся в смарт-контрактах на UTXO. Я был просто рад возможности заблокировать средства и освободить их в определенных условиях.

После assets/tokens контрактов, я понял, что это всего лишь верхушка айсберга. Я знал что это Тьюринг-полное, но после всех этих лет ограниченных Bitcoin скриптов, иметь полную силу любого произвольного алгоритма - это открыло глаза. Годы написания блокчейн-кода и очень плохие последствия с каждым багом, делают вас очень пугливым когда дело касается создания агрессивных вещей на уровне консенсуса. И это так, как должно - если не быть очень осторожным, какие нибудь очень плохие вещи могут случиться и случаются. Основание, в виде построения поверх существующей (хорошо протестированной и надежной) UTXO системы, это то, что делает CC контракты менее подверженными ужасным багам. При этом недостаточная валидация может очень просто позволить изымать средства с ненадлежащим образом разработанного CC контракта.

СС контракт выходит за рамки стандартных ограничений Bitcoin транзакции. То, что я уже написал объясняет причину этого, но сначала это было неочевидно даже для меня, поэтому возможно вы тоже упустили это. Если вы гадаете о чем это я говорю - я расскажу об ЭТОМ!

Напомню, у нас теперь есть стандартный bitcoin выход нзываемый СС выходом. Кроме того, на отдельном блокчейне может существовать до 256 различных типов выходов CC контрактов. Мы также знаем что чтобы потратить любой выход, вам нужно удовлетворить его тратящий скрипт, что в нашем случае достигается подписью и любыми ограничениями накладываемыми СС-валидацией. У нас так же есть соглашение о глобально разделенной ключевой паре, которая дает нам главный CC адрес на который можно отправить средства, наряду с конкретным CC-адресом пользователя.
Давайте вернемся к примеру с 3+2 транзакцией:

```
vin0 + vin1 + vin2 -> vout0 + vout1
```

Учитывая предыдущий параграф, попробуйте представить возможности простой 3+2 транзакции. Каждый `vin` может быть обычным `vin'ом` из глобального адреса контракта, СС адрес пользователя и `vout'ы` так же могут быть в этом диапазоне. Теоретически может существовать 257 * 257 * 257 * 257 * 257 форм 3+2 транзакций!

В реальности мы хонечно не хотим давать так много степеней свободы, так как это обеспечит нас большим числом багов!  Таким образом нам нужно сократить это число до более управляемого уровня, где для каждой из них должно быть не более 3 типов, а желательно всего 1 тип. Это сделает валидацию гораздо проще, а проще - значит лучше до тек пор пока мы не жертвуем мощью. А мы не жертвуем.

В конечном счете СС контракт заключается в том, как он ограничивает свои входы, но перед тем как он сможет их ограничить, они должны быть созданы в виде выходов. Больше об этом будет рассказано в главе про CC валидацию.



## Глава 4 - RPC расширения СС
В настоящее время CC контракты нужно интегрировать на уровне источников. Это ограничивает круг тех, кто может создавать и добавлять новые СС контракты, что с одной стороны хорошо, но в конечном итоге будет слишком строгим ограничением. В главе о привязке во время выполнения будет затронуто как выйти за ограничение привязки к исходникам, но существует и другой ключвой интерфейсный уровень - RPC.

По соглашению, каждый СС контракт добавляет связанный набор rpc вызовов в `komodo-cli`. Это не только упрощает создание транзакций СС контрактов, это так же позволит создавать dapps просто через rpc вызовы. Это потребует наличия достаточного количества уже готовых основных СС контрактов. Вместе с тем, как мы обнаруживаем новые юзкейсы которые не могут быть реализованы через rpc, создается новый CC контракт который может это обеспечить (и даже больше) и мощь rpc-уровня увеличивается. Это долгосрочный процесс.

Типичные rpc вызовы которые добавлены `<CC>address`, `<CClist>`, `<CCinfo>` возвращают различные специальные CC адреса, список экземпляров CC контрактов и информацию о каждом отдельном экзепляре СС контракта. Наряду с rpc который создает экземпляр CC и конечно вызовы для запуска экземпляра СС.
Роль rpc вызовов заключается в создании правильно подписанных `rawtransactions` которые готовы к трансляции в блокчейн. Это позволяет ограничиваться rpc вызовами не только для запуска но и для создания отдельного экземпляра СС контракта. Контракт faucet выделяется в этом, потому что имеет только один экземпляр и некоторые из этих rpc вызовов для него пропущены.

Таким образом нет ОБЯЗАТЕЛЬНЫХ rpc вызовов, есть просто разумное соглашение чтобы следовать для соответствия общей схеме.

Одна вещь которую я забыл описать, это то, как создать специальный СС адрес, и даже если это не проблема rpc. Это отчасти зависит от основных функций CC, поэтому я покажу, как это сделать здесь:

```C
const char *FaucetCCaddr = "R9zHrofhRbub7ER77B7NrVch3A63R39GuC";
const char *FaucetNormaladdr = "RKQV4oYs4rvxAWx1J43VnT73rSTVtUeckk";
char FaucetCChexstr[67] = { "03682b255c40d0cde8faee381a1a50bbb89980ff24539cb8518e294d3a63cefe12" };
uint8_t FaucetCCpriv[32] = { 0xd4, 0x4f, 0xf2, 0x31, 0x71, 0x7d, 0x28, 0x02, 0x4b, 0xc7, 0xdd, 0x71, 0xa0, 0x39, 0xc4, 0xbe, 0x1a, 0xfe, 0xeb, 0xc2, 0x46, 0xda, 0x76, 0xf8, 0x07, 0x53, 0x3d, 0x96, 0xb4, 0xca, 0xa0, 0xe9 };
```

Выше приведены параметры для faucet CC, однако каждый контракт имеет эквивалент в [CCcustom.cpp](https://github.com/jl777/komodo/tree/jl777/src/cc/CCcustom.cpp). В нижней части файла находится большой оператор switch, в котором эти значения копируются в структуру данных в памяти для каждого типа CC. Это позволяет всей кодовой базе СС иметь доступ к этим специальным адресам стандартным путем.

Чтобы получить приведенные выше значения, выполните следующие действия:

A. используйте `getnewaddress` чтобы получить новый адрес и вставьте его в строчку `<CC>Normaladdr = ""`;

B. используйте `validateaddress <новый адрес из пункта A>` для получения `pubkey`, который вставьте в строчку `<CC>hexstr[67] = ""`;

C. остановите демон [`komodod`] и запустите с параметром `-pubkey=<pubkey из пункта B>` и сделайте `<CC>address` rpc вызов. В консоли вы получите вывод hex для `privkey`, предполагая что `if ( 0 ) в Myprivkey()` включен ([CCutils.cpp](https://github.com/jl777/komodo/tree/jl777/src/cc/CCutils.cpp))

D. обновите `CCaddress` и `privkey` и не забудьте изменить параметр `-pubkey=`

Первая rpc команда которую нужно добавить - `<CC>address` и чтобы это сделать, добавьте строчку в [rpcserver.h](https://github.com/jl777/komodo/tree/jl777/src/rpcserver.h) и обновите массив команд в [rpcserver.cpp](https://github.com/jl777/komodo/tree/jl777/src/rpcserver.cpp)

В файле [rpcwallet.cpp](https://github.com/jl777/komodo/tree/jl777/src/wallet/rpcwallet.cpp) вы найдете актуальные rpc функции, найдите одну из `<CC>address`, скопируйте-вставьте, измените eval код на ваш eval код и кастомизируйте функцию. Ох, и не забудьте добавить запись в [eval.h](https://github.com/jl777/komodo/tree/jl777/src/cc/eval.h)

Теперь вы сделали собственный CC контракт, но он не будет скомпонован, поскольку вам по-прежнему нужно имплементировать его действительные функции. Это будет рассмотрено в следующих главах.



## Глава 5 - Валидация CC
СС валидация это то, в чем суть, не "фокус-покус"!

Каждый СС должен иметь собственную функцию валидации, и когда блокчейн подтверждает транзакцию, он вызовет код валидации СС. Это полностью на стороне валидации СС - будет ли валидация или нет.

Любой набор правил, о котором вы можете подумать и релизовать, может стать частью валидации. Удостоверьтесь что нет никакой двусмысленности! Убедитесь, что все транзакции, которые должны быть отклонены, фактически отклоняются.

Кроме того, убедитесь, что, любые rpc вызовы, которые создают СС транзакцию не создают ничего, что не валидируется.

На самом деле, это все что нужно сказать о валидации, которая является общей, так как это просто концепция, получающая выделенную функцию, чтобы определить, действительна ли транзакция или нет.

Для большинства первоначальных СС контрактов, я сделал работающий код для различных функций СС контрактов и добавил их вместе с созданием txid. Это делает валидацию транзакций гораздо проще, так как требуемые данные прямо здесь - в opreturn.

Вы должны быть осторожны чтобы не зайти в тупик (deadlock), так как код валидации СС вызывается когда он уже заблокирован в основном цикле протокола bitcoin. Пока первоначальные СС контракты используются как модели - вы должны избежать проблемы с тупиками.



## Chapter 6 - Faucet Example
Finally, we are ready for the first actual example of a CC contract. The faucet. This is a very simple contract and it ran into some interesting bugs in the first incarnation.

The code in [~/komodo/src/cc/faucet.cpp](https://github.com/jl777/komodo/tree/jl777/src/cc/faucet.cpp) is the ultimate documentation for it with all the details, so I will just address the conceptual issues here.

There are only 7 functions in [faucet.cpp](https://github.com/jl777/komodo/tree/jl777/src/cc/faucet.cpp), a bit over 200 lines including comments. The first three are for validation, the last four for the RPC calls to use.

```C
int64_t IsFaucetvout(struct CCcontract_info *cp,const CTransaction& tx,int32_t v)

bool FaucetExactAmounts(struct CCcontract_info cp,Eval eval,const CTransaction &tx,int32_t minage,uint64_t txfee)

bool FaucetValidate(struct CCcontract_info cp,Eval eval,const CTransaction &tx)

int64_t AddFaucetInputs(struct CCcontract_info *cp,CMutableTransaction &mtx,CPubKey pk,int64_t total,int32_t maxinputs)

std::string FaucetGet(uint64_t txfee)

std::string FaucetFund(uint64_t txfee,int64_t funds)

UniValue FaucetInfo()
```

Functions in `rpcwallet` implement:

`faucetaddress` fully implemented in [rpcwallet.cpp](https://github.com/jl777/komodo/tree/jl777/src/wallet/rpcwallet.cpp)
`faucetfund` calls `FaucetFund`
`faucetget` calls `FaucetGet`
`faucetinfo` calls `FaucetInfo`


Now you might not be a programmer, but I hope you are able to understand the above sequence. user types in a cli call, `komodo-cli` processes it by calling the RPC function, which in turn calls the function inside [faucet.cpp](https://github.com/jl777/komodo/tree/jl777/src/cc/faucet.cpp)

No magic, just simple conversion of a user command line call that runs code inside the komodod. Both the `faucetfund` and `faucetget` create properly signed rawtransaction that is ready to be broadcast to the network using the standard `sendrawtransaction` RPC. It doesnt automatically do this to allow the GUI to have a confirmation step with all the details before doing an irrevocable CC contract transaction.

`faucetfund` allows anybody to add funds to the faucet

`faucetget` allows anybody to get 0.1 coins from the faucet as long as they dont violate the rules.

And we come to what it is all about. The rules of the faucet. Initially it was much less strict and that allowed it to be drained slowly, but automatically and it prevented most from being able to use the faucet.

To make it much harder to leech, it was made so each `faucetget` returned only 0.1 coins (down from 1.0) so it was worth 90% less. It was also made so that it had to be to a fresh address with less than 3 transactions. Finally each txid was constrained to start and end with 00! This is a cool trick to force usage of precious CPU time (20 to 60 seconds depending on system) to generate a valid txid. Like PoW mining for the txid and I expect other CC contracts to use a similar mechanism if they want to rate limit usage.

Combined, it became such a pain to get 0.1 coins, the faucet leeching problem was solved. It might not seem like too much trouble to change an address to get another 0.1 coins, but the way things are setup you need to launch the `komodod -pubkey=<your pubkey>` to change the pubkey that is active for a node. That means to change the pubkey being used, the `komodod` needs to be restarted and this creates a lot of issues for any automation trying to do this. Combined with the PoW required, only when 0.1 coins becomes worth a significant effort will faucet leeching return. In that case, the PoW requirement can be increased and coin amount decreased, likely with a faucet2 CC contract as I dont expect many such variations to be needed.





## Chapter 7 - Rewards Example

## Chapter 8 - Assets Example

## Chapter 9 - Dice Example

## Chapter 10 - Lotto Example

## Chapter 11 - Channels Example



## Chapter 12 - Limitless Possibilities
As can be seen, CC contracts can do a wide range of things and since they are Turing complete, we know that this is true. However, what is more important is the added security gained from using a UTXO based system. While in some ways it is more complex to have to deal with UTXO, as can be seen by the above examples, it is either solved and made invisible at the RPC level, or actually used as part of the solution.

Being UTXO based, automatically builds in a rate limit to how many tx per block a specific CC contract can do. The state advancing by one transaction at a time is another means that rate limits. Since more UTXO can be made available to increase capacity, it actually offers a way for managing load.

I believe I have made one of the first operational UTXO smart contracts, CC or otherwise and hope that there will be many more developers joining forces to create more foundational CC contracts. Feel free to contact me for feedback on the type of CC contract you want to make. I have not documented all my notes and it could well be I already sort of know how to implement what your want your CC contract to do. Just only so many I can actually make time to code and debug.

Our testing cycle went a lot faster than expected as the bugs found were few and far between. Considering the scope of the assets CC and the realtime response aspects of dice CC, this was quite unexpected. I can only attribute it to the fact that CC validation is just the final validation on top of all the standard bitcoin protocol validations. Not having to worry about double spends is sure a nice luxury, though dont get too complacent about chain rewrites! It is possible to wait for information to be divulged and then reorg the chain to take advantage of this knowledge in a chain which is rewound.

Yes, blockchains are complicated.

## Chapter 13 - Different Languages
The current codebase is integrated into the komodod codebase, which is C/C++. However, it is possible to use different languages and integrate into the C/C++ as zcash has shown by using the rust language for some parts of the `zcashd`.

I think any language that is compiled and can create a linkable library while being able to call and be called by C/C++ functions can be used. If you are able to make such a language binding for a simple CC contract like faucet, this will be good for a 777 KMD bounty. Of course, you need to be the first to submit a properly working pull request.


## Chapter 14 - Runtime Bindings
Once build time linking works, then it is one step away from being able to do runtime linking, ie. dynamically linked libraries. There will be some work required to prevent duplication of eval codes and making sure it is a valid version of the CC contract plugin, but these are issues that have been solved before and I dont see any reason they cant be solved for CC contracts.

This would open up the door for quite an interesting ecosystem of CC plugins that blockchains can subscribe to.

## Chapter 15 - RPC based dapps
Ultimately, I expect there to be so many new RPC calls (one set from each CC contract), that virtually any dapp can be made with RPC calls. We are just at the beginning now, but it is just a matter of time when we get there.

For now, we just need to keep listening to what the market wants as far as dapps go. Then make a new CC contract that enables doing as many of those as possible.

Repeat...

Imagine the scope that will exist after a year or two of continuous new CC contracts being created, along with all the RPC based dapps. I have seen some automatic GUI generators and it could be that for most cases, there can be a special GUI that not only create the dapp's GUI, but also all the RPC calls that are needed to make it work the way it is customized.

This codebase and tools in between the GUI and the RPC level will be a very good area for new initiatives.

---------

## Conclusion
I hope this document has helped you understand what a Komodo utxo based CC contract is and how it is different from the other smart contracts. If you are now able to dive into the cc directory and start making your own CC contract, then I am very happy!
