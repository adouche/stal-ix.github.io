# stal/IX blog

> About building systems in a different way, static linking, and much more!<br>
> And about the stal/IX, of course, from benevolent dictator (@pg83).

## Long read. #bootstrap
<sup> October 2, 2021 </sup>

Я думаю, не секрет, что мне очень интересны системы сборки, в самом разнообразном виде. Настолько, что у меня есть:

1) Своя система сборки для С++ кода
* https://github.com/pg83/zm
* https://github.com/pg83/zm/blob/master/tp/libs/python/src/z.make

2) И своя система сборки OSS пакетов. Для Darwin и для Linux, для последнего это не только система пакетов, но и полноценный дистрибутив:
* https://github.com/pg83/mix
* https://github.com/pg83/mix/blob/main/pkgs/dev/lang/python3/package.sh (самый большой сборочный пакет, потому что собрать статически слинкованный Питон - совершенно нетривиальная задача)

Об этих проектах я еще буду рассказывать, особенно про второй. Сейчас я их привел для понимания того, что тема сборки, дистросроения, бутстрапа, мне очень интересны.

Сегодня я хочу поговорить про задачу бутстрапа.

Что это такое? Представьте себе, что вы попали на необитаемый остров, у вас есть пустой компьютер(без ОС), и компакт-диск с исходниками программ. Нужно из этого получить рабочую ОС, чтобы на необитаемом острове было не так скучно(стюардесса на необитаемый остров с нами не попала).

Эта задача имеет следующие аспекты:

1) Исторический. Как вообще проходил путь от компьютера с перфокартами до современных рабочих станций и планшетов?
2) Сугубо практический - как развернуть дистрибутив, имея максимально малый возможный бинарный seed. Вот очень интересное исследование, как проложить путь от минимального гипервизора в 200 инструкций до современного дистрибутива Linux - https://github.com/fosslinux/live-bootstrap/blob/master/parts.rst
3) Безопасность. Воспроизводимая сборка с минимальным бинарным seed нужна, чтобы быть уверенным, что в софте нет закладок. Например, небезызвестная атака Томпсона: https://www.cs.cmu.edu/~rdriley/487/papers/Thompson_1984_ReflectionsonTrustingTrust.pdf . Очень смешной вариант - https://www.quora.com/What-is-a-coders-worst-nightmare/answer/Mick-Stute?share=1 Да и просто понимать, что твоя сборка Alacritty собрана из исходников с github, и не содержит в себе трояна или какой другой жести, очень ценно.

Я стараюсь не держать у себя софта, для которого нельзя протянуть вот такую вот цепочку from ground truth. 

В интернетах есть несколько community, которых эта задача тоже беспокоит:

1) https://bootstrappable.org/
2) https://guix.gnu.org/ (очень сильно упарываются по bootstrap)
3) https://nixos.org/ (в меньшей степени, не стесняются использовать бинарные сборки Rust, например)

Товарищи из Guix даже разработали пару специализированных языков для первоначального bootstrap(https://www.gnu.org/software/mes/, https://guix.gnu.org/blog/2019/guix-reduces-bootstrap-seed-by-50/) У guix, правда, есть другие свои заморочки, поэтому не могу его рекомендовать для повседневного использования. Nix в этом отношении интереснее, я его использовал как пакетный менеджер для любого Linux, Darwin, пока не завершил задачу bootstrap своего дистрибутива.

Что стоит еще рассказать про тему bootstrap:
* Далеко не все языки можно bootstrap. Это значит, что для некоторых языков нужен большой изначальный blob неизвестного качества. Я такие языки стараюсь не использовать(https://elephly.net/posts/2017-01-09-bootstrapping-haskell-part-1.html).
* Java бутстрапнули совсем недавно(https://bootstrappable.org/projects/jvm-languages.html). Систему сборки для нее(gradle) до сих пор не.
* Rust, с моей точки зрения, с натяжкой bootstrapable. Потому что: #mrustc
1) Для сборки n-ой версии нужна n-1 версия. Это очень сильно увеличивает длину цепочки bootstrap. Порядка 20 сборок на текущий момент?
2) Исходники первых версий на standard ml лежат неизвестно где. Bootstrap возможен только с сиспользованием стороннего продукта(https://github.com/thepowersgang/mrustc)
3) На Apple M1 цепочку пока повторить не получается
* У Go с bootstrap все хорошо, авторы поддерживают версию 1.4, последнюю на С. С ее помощью можно собрать современный Go. Так же существует gccgo.
* У Lisp все тоже очень хорошо. Скажем, цепочку от интерпретируемого ECL до компилируемого SBCL на Apple M1 я протянул за час.