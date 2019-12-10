
# Reverse engineering
https://ctf.school/challenges

- [ Snake ](#snake)
- [ Robot ](#robot)
- [ Bin ](#bin)

<a name="snake"></a>
## Snake
#### Наша компания всегда пытается обойти любую защиту программ. Помоги нам разобраться с Питоном в этот раз

Что мы имеем? Файл task.pyc
```bash
$ cat task.pyc
B
8��]D�@s(ddlZdZdd�Zedkr$ed�dS)�NZw3l1_pycCs6d}|t|}td�}||kr*td�td�dS)Nzth0n}zHello! Enter flag to check: zNice!zNope.)�  flg_part2�input�print)Z flg_part1Z      flg_part3ZflgZflg1�r�task.py�mais


r__main__zflag{)�stringrr__name__rrrr<module>s  
```

Видим:
- Hello! Enter flag to check
- Nice!
- Nope

Значит внутри программа проверки на валидность введённого с stdin флага<br>
Осталось достать флаг :smirk:.

Откроем файл в текстовом редакторе:
```text
420d 0d0a 0000 0000 38a3 da5d 4401 0000
e300 0000 0000 0000 0000 0000 0002 0000
0040 0000 0073 2800 0000 6400 6401 6c00
5a00 6402 5a01 6403 6404 8400 5a02 6503
6405 6b02 7224 6502 6406 8301 0100 6401
5300 2907 e900 0000 004e 5a07 7733 6c31
5f70 7963 0100 0000 0000 0000 0400 0000
0200 0000 4300 0000 7336 0000 0064 017d
017c 0074 0017 007c 0117 007d 0274 0164
0283 017d 037c 027c 036b 0272 2a74 0264
0383 0101 006e 0874 0264 0483 0101 0064
0053 0029 054e 7a05 7468 306e 7d7a 1c48
656c 6c6f 2120 456e 7465 7220 666c 6167
2074 6f20 6368 6563 6b3a 207a 054e 6963
6521 7a05 4e6f 7065 2e29 03da 0966 6c67
5f70 6172 7432 da05 696e 7075 74da 0570
7269 6e74 2904 5a09 666c 675f 7061 7274
315a 0966 6c67 5f70 6172 7433 5a03 666c
675a 0466 6c67 31a9 0072 0500 0000 fa07
7461 736b 2e70 79da 046d 6169 6e08 0000
0073 0c00 0000 0001 0401 0c01 0801 0801
0a02 7207 0000 00da 085f 5f6d 6169 6e5f
5f7a 0566 6c61 677b 2904 da06 7374 7269
6e67 7202 0000 0072 0700 0000 da08 5f5f
6e61 6d65 5f5f 7205 0000 0072 0500 0000
7205 0000 0072 0600 0000 da08 3c6d 6f64
756c 653e 0200 0000 7308 0000 0008 0204
0408 0a08 01
```

Файлы .pyc содержат байт-код, который является исходным кодом для интерпретатора Python. <br>
Этот код затем выполняется виртуальной машиной Python. <br>
(https://stackoverflow.com/questions/2998215/if-python-is-interpreted-what-are-pyc-files)

Получить подобный код можно с помощью `python -m compileall ...`.

pyc-файл состоит из:
* четырёхбайтового магического номера;
* четырёхбайтовой метки времени;
* четырёх байт хранящих размер исходного файла;
* сериализованного объекта кода.

Магический номер - это два байта уникальных для каждой версии интерпретатора и два байта 0d0a. <br>
Байты 0d0a - это символ возврата каретки и перевода строки, защищающие файл от повреждения в случае редактирования в текстовом режиме. <br>
(https://ru.stackoverflow.com/questions/797871/Как-устроены-pyc)

Попробуем просто натравить интерпретатор на данный файл:
```bash
$ python tasks/task.pyc 
RuntimeError: Bad magic number in .pyc file

$ python --version
Python 2.7.16
```

Так-с, Python 2.7 ему не угодил. Попробуем другой
```bash
$ python3 --version
Python 3.7.2

$ python3 tasks/task.pyc 
Hello! Enter flag to check: 123
Nope.
```

Ага, значит это байт-код программы на Python 3. <br>
И действительно, магическое число совпадает с тем, что мы видели через текстовый редактор
```
$ python3
Python 3.7.2 (v3.7.2:9a3ffc0492, Dec 24 2018, 02:44:43) 
[Clang 6.0 (clang-600.0.57)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> import importlib
>>> importlib.util.MAGIC_NUMBER.hex()
'420d0d0a'
```

Что нам это дает? Ничего :) <br>
Знание, на каком именно Python написана программа, бесполезно.<br>
Если мы не хотим, конечно, подбирать флаг запуская программу раз за разом.

Нам поможет декомпиляция. Подобного модуля у Python нет, но есть библиотеки:
- https://pypi.org/project/uncompyle6/
- https://github.com/Mysterie/uncompyle2
- https://github.com/zrax/pycdc

Их надо устанавливать, вникать, может просто есть онлайн-декомпилятор?

"python online decompiler" -> Google -> https://python-decompiler.com/
 
Загружаем наш файл и получаем исходник :smiling_imp::
```python
import string
flg_part2 = 'w3l1_py'

def main(flg_part1):
    flg_part3 = 'th0n}'
    flg = flg_part1 + flg_part2 + flg_part3
    flg1 = input('Hello! Enter flag to check: ')
    if flg == flg1:
        print('Nice!')
    else:
        print('Nope.')


if name == '__main__':
    main('flag{')
```

Несложным образом узнаем наш флаг:
```python
flag = 'flag{' + 'w3l1_py' + 'th0n}' = 'flag{w3l1_pyth0n}'
```

Проверяем через исходный .pyc:
```bash
$ python3 task.pyc 
Hello! Enter flag to check: flag{w3l1_pyth0n}
Nice!
```

<a name="robot"></a>
## Robot
#### В этот раз нам попалось странное приложение, помоги найти пароль для входа.

Что мы имеем? Файл app.apk

APK (англ. Android Package) — формат архивных исполняемых файлов-приложений для Android.

Но мы не будем пытаться поставить это приложение на мобильное устройство, а сразу ищем декомпилятор.

"apk online decompiler" -> Google -> http://www.javadecompilers.com

Онлайн-декомпиляторов гораздо больше, чем для Python, загружаем в первый попавшийся нашу apk-шку.

Видим две директории:
```text
app.apk
├── resources
└── sources
```

Я не мобильный разработчик, но что-то мне подсказывает, что *sources* ближе к исходному коду, чем *resources*:
```text
sources
├── androidx
├── android
├── org
├── kotlinx
├── kotlin
└── p006ru
```

Во всех директориях кроме последней лежат файлы, слабо напоминающие код приложения.<br>
Видимо, там third-party и стандартная библиотека. Остановимся на *p006ru*:
```text
p006ru
└── nickmiller
    └── lozawetdream
        └── MainActivity$onCreate$1.java
        ├── MainActivity.java
        ├── MainActivityKt.java
        ├── BuildConfig.java
        ├── C0285R.java
        ├── Foo.java
        └── MainActivityKt$a$1.java
```

Уже гораздо лучше! Давайте разбираться.

**MainActivity$onCreate$1.java**
```java
// ...

final class MainActivity$onCreate$1 implements OnClickListener {
    final /* synthetic */ MainActivity this$0;

    MainActivity$onCreate$1(MainActivity mainActivity) {
        this.this$0 = mainActivity;
    }

    public final void onClick(View it) {
        TextView textView = (TextView) this.this$0._$_findCachedViewById(C0285R.C0287id.output);
        Intrinsics.checkExpressionValueIsNotNull(textView, "output");
        EditText editText = (EditText) this.this$0._$_findCachedViewById(C0285R.C0287id.key);
        Intrinsics.checkExpressionValueIsNotNull(editText, "key");
        textView.setText(MainActivityKt.m35a(editText.getText().toString()));
    }
}
```
Что-то куда-то выводим по клику, неинтересно.

**MainActivity.java**
```
// ...

public final class MainActivity extends AppCompatActivity {
    private HashMap _$_findViewCache;

    public void _$_clearFindViewByIdCache() {
        // ...
    }

    public View _$_findCachedViewById(int i) {
        // ...
    }

    /* access modifiers changed from: protected */
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView((int) C0285R.layout.activity_main);
        ((Button) _$_findCachedViewById(C0285R.C0287id.checkBtn)).setOnClickListener(new MainActivity$onCreate$1(this));
    }
}
```
Похоже на код создания главного экрана. Видим инициализацию класса *MainActivity$onCreate$1* из предыдущего файла и его привязку на событие клика по кнопке. Неинтересно.

**BuildConfig.java**
```java
package p006ru.nickmiller.lozawetdream;

/* renamed from: ru.nickmiller.lozawetdream.BuildConfig */
public final class BuildConfig {
    public static final String APPLICATION_ID = "ru.nickmiller.lozawetdream";
    public static final String BUILD_TYPE = "debug";
    public static final boolean DEBUG = Boolean.parseBoolean("true");
    public static final String FLAVOR = "";
    public static final int VERSION_CODE = 1;
    public static final String VERSION_NAME = "1.0";
}
```
Обычный конфиг сборки, флагов не наблюдается...

**C0285R.java**<br>
Огромный файл с константами, видимо, что-то служебное. Нам явно не сюда.

**Foo.java**
```java
package p006ru.nickmiller.lozawetdream;

/* renamed from: ru.nickmiller.lozawetdream.Foo */
public class Foo {
    public static String bar(String s, String key) {
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < s.length(); i++) {
            sb.append((char) (s.charAt(i) ^ key.charAt(i % key.length())));
        }
        return sb.toString();
    }

    public static byte[] bar2(byte[] input, byte[] secret) {
        byte[] output = new byte[input.length];
        if (secret.length != 0) {
            int spos = 0;
            for (int pos = 0; pos < input.length; pos++) {
                output[pos] = (byte) (input[pos] ^ secret[spos]);
                spos++;
                if (spos >= secret.length) {
                    spos = 0;
                }
            }
            return output;
        }
        throw new IllegalArgumentException("empty security key");
    }
}
```
Интересно, но нужно поискать вызовы bar и bar2. Выглядит как отвлекающий манёвр.

**MainActivityKt.java**
```java
// ...

final class MainActivityKt$a$1 extends Lambda implements Function1<Boolean, String> {

    /* renamed from: $m */
    final /* synthetic */ String f45$m;

    MainActivityKt$a$1(String str) { // Принимаем в конструкторе строку
        this.f45$m = str;            // Сохраняем её в классе
        super(1);
    }

    public /* bridge */ /* synthetic */ Object invoke(Object obj) {
        return invoke(((Boolean) obj).booleanValue());
    }

    public final String invoke(boolean it) {
        if (!it) {
            return "Try again";
        }
        StringBuilder sb = new StringBuilder();
        sb.append("flag{");
        sb.append(this.f45$m);  // Оборачиваем строку, сохраненную в конструкторе
        sb.append('}');
        return sb.toString();
    }
}
```
Так, уже интереснее. Мы видим, что можно создать объект класса **MainActivityKt$a$1** от строки. <br>
Затем вызвать у объекта метод **invoke** и получить нашу строку, обернутую в "flag{ }" или "Try again" :smirk:. <br>
Осталось найти использование всего этого дела и разобраться, что к чему...

**MainActivityKt$a$1.java**
```
// ...

public final class MainActivityKt {

    /* renamed from: h */
    private static final byte[] f44h = {102, 97, 50, 98, 102, 54, 52, 54, 101, 52, 57, 97, 98, 53, 101, 53, 54, 102, 50, 98, 55, 52, 52, 56, 48, 98, 97, 54, 49, 48, 49, 55};

    /* renamed from: a */
    public static final String m35a(String b) {
        Intrinsics.checkParameterIsNotNull(b, "b");
        String m = magic(b);
        return resolve(m, new MainActivityKt$a$1(m));
    }

    public static final String resolve(String $this$resolve, Function1<? super Boolean, String> a) {
        Intrinsics.checkParameterIsNotNull($this$resolve, "$this$resolve");
        Intrinsics.checkParameterIsNotNull(a, "a");
        byte[] bytes = $this$resolve.getBytes(Charsets.UTF_8);
        Intrinsics.checkExpressionValueIsNotNull(bytes, "(this as java.lang.String).getBytes(charset)");
        return (String) a.invoke(Boolean.valueOf(Arrays.equals(bytes, f44h)));
    }

    public static final String magic(String s) {
        Intrinsics.checkParameterIsNotNull(s, "s");
        MessageDigest instance = MessageDigest.getInstance(new String("MD5".getBytes(), Charsets.UTF_8));
        MessageDigest $this$apply = instance;
        byte[] bytes = s.getBytes(Charsets.UTF_8);
        Intrinsics.checkExpressionValueIsNotNull(bytes, "(this as java.lang.String).getBytes(charset)");
        $this$apply.update(bytes);
        byte[] $this$fold$iv = instance.digest();
        Intrinsics.checkExpressionValueIsNotNull($this$fold$iv, "MessageDigest.getInstanc…ay()) }\n        .digest()");
        StringBuilder sb = new StringBuilder();
        int length = $this$fold$iv.length;
        for (int i = 0; i < length; i++) {
            StringBuilder st = sb;
            String h = Integer.toHexString($this$fold$iv[i] & UByte.MAX_VALUE);
            Intrinsics.checkExpressionValueIsNotNull(h, "Integer.toHexString(0xFF and b.toInt())");
            while (h.length() < 2) {
                StringBuilder sb2 = new StringBuilder();
                sb2.append('0');
                sb2.append(h);
                h = sb2.toString();
            }
            st.append(h);
            Intrinsics.checkExpressionValueIsNotNull(st, "st.append(h)");
            sb = st;
        }
        String sb3 = sb.toString();
        Intrinsics.checkExpressionValueIsNotNull(sb3, "MessageDigest.getInstanc…d(h)\n        }.toString()");
        return sb3;
    }
}
```

Методы **magic** и **resolve** вызываются в методе **m35a**. Но где используется **m35a**? <br>
Несостыковочка... Ещё раз пробегаемся глазами по файлам и видим в **MainActivity$onCreate$1.java**:
```java
textView.setText(MainActivityKt.m35a(editText.getText().toString()));
```
А не флаг ли случаем (или фраза "Try again") выводится на экран по клику?

Посмотрим на методы последнего класса:
```java
private static final byte[] f44h = {102, 97, 50, 98, 102, 54, 52, 54, 101, 52, 57, 97, 98, 53, 101, 53, 54, 102, 50, 98, 55, 52, 52, 56, 48, 98, 97, 54, 49, 48, 49, 55};

public static final String m35a(String b) {
    // ...
    String m = magic(b);                          // Какая-то магия
    return resolve(m, new MainActivityKt$a$1(m)); // Инициализация нашего "оборачивателя во 'flag{ }'"
}

public static final String resolve(String $this$resolve, Function1<? super Boolean, String> a) {
    // ...
    byte[] bytes = $this$resolve.getBytes(Charsets.UTF_8);
    // ...
    // Вернуть магически полученную строку, обернутую во "flag{ }",
    // если параметр invoke == True
    return (String) a.invoke(Boolean.valueOf(
        // Сравнение байтов строки со статично определенным массивом байт
        Arrays.equals(bytes, f44h)
    ));
}

public static final String magic(String s) {
    // Много букв
}
```
Что мы видим?
- Получение строки магическим образом
- Сравнение получившейся строки и массива байт **f44h**
- Если совпадают, то возвращается строка, обернутая в "flag{ }" (см. **MainActivityKt.java**)
- Дальнейший вывод на экран флага

Что мы можем сделать?
- Разобраться в коде функции **String magic(String s)** (что я сначала и начал делать :smile:)
- Перевести **f44h** в ASCII, так как уж слишком заманчиво эти байты выглядят <br>
и по сути они и выводятся на экран в составе "flag{ }"

Второе явно приятнее, воспользуемся Python:
```python
print(
    ''.join(map(chr, [
        102, 97, 50, 98, 102, 54, 52, 54, 101, 52, 57, 97, 98, 53, 101, 53,
        54, 102, 50, 98, 55, 52, 52, 56, 48, 98, 97, 54, 49, 48, 49, 55
    ]))
) # fa2bf646e49ab5e56f2b74480ba61017
```

И наш флаг - **flag{fa2bf646e49ab5e56f2b74480ba61017}** :smiling_imp:!

<a name="bin"></a>
## Bin
#### В этот раз нам попался бинарный файл. Задача все та же, достать секретный пароль.

Не мудрствуя лукаво: "disassembler online" -> Google -> https://onlinedisassembler.com/odaweb/

Загружаем файл и смотрим ассемблерный код. Что у нас по строкам?
```text
Address	String
0x2a8	/lib64/ld-linux-x86-64.so.2
0x5d9	libstdc++.so.6
0x5e8	__gmon_start__
0x5f7	_ITM_deregisterTMCloneTable
0x613	_ITM_registerTMCloneTable
0x62d	_ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE7compareEPKc
0x670	_ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE6lengthEv
0x6b0	_ZNSaIcED1Ev
0x6bd	_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEC1Ev
0x6f7	_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6_
0x732	_ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE6substrEmm
0x773	_ZSt3cin
0x77c	_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEC1EPKcRKS3_
0x7bd	_ZNSt8ios_base4InitD1Ev
0x7d5	_ZNSolsEPFRSoS_E
0x7e6	__gxx_personality_v0
0x7fb	_ZNSaIcEC1Ev
0x808	_ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE5c_strEv
0x847	_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEC1ERKS4_
0x885	_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc
0x8bd	_ZNSt8ios_base4InitC1Ev
0x8d5	_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev
0x90f	_ZSt4cout
0x919	_ZStrsIcSt11char_traitsIcESaIcEERSt13basic_istreamIT_T0_ES7_RNSt7__cxx1112basic_stringIS4_S5_T1_EE
0x97c	libgcc_s.so.1
0x98a	_Unwind_Resume
0x999	libc.so.6
0x9a3	__cxa_atexit
0x9b0	__cxa_finalize
0x9bf	strcmp
0x9c6	__libc_start_main
0x9d8	GCC_3.0
0x9e0	GLIBC_2.2.5
0x9ec	CXXABI_1.3
0x9f7	GLIBCXX_3.4
0xa03	GLIBCXX_3.4.21
0x11f7	u/UH
0x1296	_eeef
0x1351	k14sf
0x145c	_172f
0x1564	feeff
0x1992	[]A\A]A^A_
0x2005	flag{
0x200b	flag{soo_simple}
0x201c	Hello! Enter flag to check:
0x2039	Nice! Correct!
0x2048	Nope! Wrong!
0x2147	;*3$"
0x2195	zPLR
```
libc, строковые функции, так-так...

```text
0x11f7	u/UH
0x1296	_eeef
0x1351	k14sf
0x145c	_172f
0x1564	feeff
0x1992	[]A\A]A^A_
0x2005	flag{
0x200b	flag{soo_simple}
0x201c	Hello! Enter flag to check:
0x2039	Nice! Correct!
0x2048	Nope! Wrong!
0x2147	;*3$"
0x2195	zPLR
```
Похоже эта программа тоже проверяет вводимый флаг и выводит "Nice! Correct!" или "Nope! Wrong!".

Ответ `flag{soo_simple}` система, очевидно, не принимает. Ну ладно :cry: <br>
Отметим наличие кусочка "flag{" и каких-то разбросанных текстовых фрагментов.

Посмотрим, какие у нас здесь функции:
```text
Address	Type	Name
0x1000	t	_init
0x1020	t	.plt
0x1030	t	func_00001030
0x1040	t	func_00001040
0x1050	t	func_00001050
0x1060	t	func_00001060
0x1070	t	func_00001070
0x1080	t	func_00001080
0x1090	t	func_00001090
0x10a0	t	func_000010a0
0x10b0	t	func_000010b0
0x10c0	t	func_000010c0
0x10d0	t	func_000010d0
0x10e0	t	func_000010e0
0x10f0	t	func_000010f0
0x1100	t	func_00001100
0x1110	t	func_00001110
0x1120	t	func_00001120
0x1130	t	func_00001130
0x1140	t	.plt.got
0x1150	t	.text
0x1180	t	deregister_tm_clones
0x11b0	t	register_tm_clones
0x11f0	t	__do_global_dtors_aux
0x1230	t	frame_dummy
0x1235	T	_Z6check5NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
0x12e7	T	_Z6check4NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
0x13fb	T	_Z6check3NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
0x1503	T	_Z6check2NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
0x160b	T	_Z6check1NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
0x16c0	T	_Z5checkNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
0x173e	T	main
0x18a9	t	_Z41__static_initialization_and_destruction_0ii
0x18f2	t	_GLOBAL__sub_I_d
0x1940	T	__libc_csu_init
0x19a0	T	__libc_csu_fini
0x19a4	t	.fini
0x3db0	t	__init_array_start
0x3dc0	t	__init_array_end
```
Куча небольших подпрограмм **func_***, часть из которых наверняка служебные.<br>
**\*basic_string\*char_traits** - скорее всего набор конструкторов строк.<br>
И остальные вспомогательные процедуры.

Функции 0x1235-0x16c0 намекают, что флаг получается непростым способом с <br>
использованием каких-то строковых функций, ну что ж...

Посмотрим на граф вызовов:
[call_graph.png](img/call_graph.png)

Видим наш "flag{simple}}", но дальнейшего использования вроде бы нет:
```text
.text:0000175b  lea 0x8a9(%rip),%rsi  # 0x0000200b
```

Видим вывод на экран приглашения "0x201c	Hello! Enter flag to check:":
```text
.text:00001776  lea 0x89f(%rip),%rsi   # 0x0000201c
.text:0000177d  lea 0x293c(%rip),%rdi  # 0x000040c0 <_ZSt4cout>
.text:00001784  callq 0x00001080
 ```
 
Видим ввод флага:
```text
.text:0000179c  lea 0x2a3d(%rip),%rdi  # 0x000041e0 <_ZSt3cin>
.text:000017a3  callq 0x000010c0
```

Видим вызов строковой функции и дальнейшую проверку результата (**test**):
```text
.text:000017c2  callq 0x000016c0 <_Z5checkNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE>
.text:000017c7  mov %eax,%ebx
.text:000017c9  lea -0x30(%rbp),%rax
.text:000017cd  mov %rax,%rdi
.text:000017d0  callq 0x00001060
.text:000017d5  test %bl,%bl
```

И затем "прыжок" на вывод "0x2039	Nice! Correct!":
```text
.text:000017d9  lea 0x859(%rip),%rsi # 0x00002039
.text:000017e0  lea 0x28d9(%rip),%rdi # 0x000040c0 <_ZSt4cout>
.text:000017e7  callq 0x00001080
```

или на вывод "0x2048	Nope! Wrong!":
```text          
.text:00001803  lea 0x83e(%rip),%rsi # 0x00002048
.text:0000180a  lea 0x28af(%rip),%rdi # 0x000040c0 <_ZSt4cout>
.text:00001811  callq 0x00001080
```

Общая картина программы более-менее ясна.

Скорее всего конструирование флага происходит при вызове
```text
.text:000017c2  callq 0x000016c0 <_Z5checkNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE>
``` 

Посмотрим эту функцию подробнее:
[call_graph1.png](img/call_graph1.png)

Видим вызов какой-то функции и сравнение с 0x19 (25):
```text
.text:000016d4  callq 0x00001100
.text:000016d9  cmp $0x19,%rax
.text:000016dd  setne %al
.text:000016e0  test %al,%al
```

И при положительном результате вызов **_Z6check1NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE**.

Перейдём к ней:
[call_graph2.png](img/call_graph2.png)

Много букв, сложно, но, мы видим фигурирование флага и вызов следующей строковой функции
```text
.text:00001636  lea    0x9c8(%rip),%rsi        # 0x00002005
.text:0000163d  mov    %rax,%rdi
.text:00001640  callq  0x00001907 <_ZSteqIcSt11char_traitsIcESaIcEEbRKNSt7__cxx1112basic_stringIT_T0_T1_EEPKS5_>
```

Напомню, что 0x00002005 - это как раз наш кусочек
```text
.rodata:00002005  0x66 'f'
.rodata:00002006  0x6c 'l'
.rodata:00002007  0x61 'a'
.rodata:00002008  0x67 'g'
.rodata:00002009  0x7b '{'
.rodata:0000200a  0x00
```

Дальше код повторяется:
- какие-то ассемблерные операции
- вызов очередной строковой функции

Понятно, что происходит конструирование флага, но разобраться в ассемблере не так просто :unamused:

А что если есть декомпилятор, если не в С++, то хотя бы в Си?

"convert asm to c" -> Google -> https://retdec.com/

1) Следуем инструкциям по установке: https://github.com/avast/retdec
2) Скачиваем с сайта дизасемблированный код в виде файла
2) Натравливаем на файл retdec
```bash
$ python $RETDEC_INSTALL_DIR/bin/retdec-decompiler.py task.txt
```

Получаем task.c, не идеальный код, но жить можно:
```с
// ...
// Address range: 0x173e - 0x18a9
int main(int argc, char ** argv) {
    // 0x173e
    int64_t v1; // bp-136
    int64_t v2 = &v1; // 0x1743
    int64_t v3; // bp-57
    function_1130(&v3);
    int64_t v4; // bp-104
    function_10d0(&v4, "flag{soo_simple}", &v3);
    function_10b0(&v3);
    function_1080(&g5, "Hello! Enter flag to check: ");
    function_10e0(&v1);
    function_10c0(&g6, &v1);
    int64_t v5; // bp-56
    function_1050(&v5, v2, v2);
    int64_t v6 = _Z5checkNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(&v5); // 0x17c2
    function_1060(&v5);
    if ((char)v6 == 0) {
        int64_t v7 = function_1080(&g5, "Nope! Wrong!"); // 0x1811
        function_10a0(v7, *(int64_t *)0x3fd0);
    } else {
        int64_t v8 = function_1080(&g5, "Nice! Correct!"); // 0x17e7
        function_10a0(v8, *(int64_t *)0x3fd0);
    }
    // 0x182b
    function_1060(&v1);
    function_1060(&v4);
    return 0;
}
// ...
```
Структура программы соответствует той, что мы видели в ассемблере:
- обманный манёвр в виде инициализации "flag{soo_simple}"
- получение флага через **_Z5checkNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE**
- вывод Wrong / Correct

Запомним, что при правильном результате функция получения флага возаращает не ноль.

Посмотрим её:
```c
// Address range: 0x1100 - 0x1106
int64_t function_1100(int64_t a1) {
    // 0x1100
    return _ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE6lengthEv();
}

// Address range: 0x16c0 - 0x173e
// Demangled:     check(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
int64_t _Z5checkNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(int64_t * a1) {
    int64_t v1 = (int64_t)a1; // 0x16c9
    int64_t result; // 0x1705
    if (function_1100(v1) == 25) {  // Длина флага - 25 символов!
        // 0x16eb
        int64_t v2; // bp-56
        function_1050(&v2, v1, v1);
        result = _Z6check1NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(&v2);
        function_1060(&v2);
    } else {
        result = 0;
    }
    // 0x1719
    return result;
}
```
А вот и наше число 25, которое мы видели в ассемблерном аналоге этой функции. <br>
Судя по _ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE6**length**Ev в **function_1100** <br>
в данном месте идет проверка на длину входной строки.

Окей, значит наш флаг состоит из 25 символов.

Что дальше? Дальше набор из последовательного вызова строковых функций, который мы не осилили в ассемблере:
```c
// Address range: 0x160b - 0x16c0
// Demangled:     check1(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
int64_t _Z6check1NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(int64_t * a1) {
    int64_t v1 = (int64_t)a1; // 0x1614
    int64_t v2; // bp-88
    function_1090(&v2, v1, 0, 5);
    int64_t v3 = _ZSteqIcSt11char_traitsIcESaIcEEbRKNSt7__cxx1112basic_stringIT_T0_T1_EEPKS5_(&v2, "flag{"); // 0x1640
    int64_t result; // rbx
    if ((char)v3 == 0) {
        // 0x1679
        result = 0;
    } else {
        // 0x1649
        int64_t v4; // bp-56
        function_1050(&v4, v1, v1);
        result = _Z6check2NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(&v4);
        function_1060(&v4);
    }
    // 0x167e
    function_1060(&v2);
    return result;
}

// Address range: 0x1503 - 0x160b
// Demangled:     check2(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
int64_t _Z6check2NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(int64_t * a1) {
    int64_t v1 = (int64_t)a1; // 0x150c
    int64_t v2; // bp-104
    function_1090(&v2, v1, 5, 5);
    int64_t str2 = g10 - 1 & -16; // 0x1552
    *(int32_t *)str2 = 0x66656566;
    *(int16_t *)(str2 + 4) = 97;
    int64_t str = function_1040(&v2); // 0x1575
    int64_t result; // rbx
    if (strcmp((char *)str, (char *)str2) != 0) {
        // 0x15c5
        result = 0;
    } else {
        // 0x1595
        int64_t v3; // bp-72
        function_1050(&v3, v1, v1);
        result = _Z6check3NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(&v3);
        function_1060(&v3);
    }
    // 0x15ca
    function_1060(&v2);
    return result;
}

// Address range: 0x13fb - 0x1503
// Demangled:     check3(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
int64_t _Z6check3NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(int64_t * a1) {
    int64_t v1 = (int64_t)a1; // 0x1404
    int64_t v2; // bp-104
    function_1090(&v2, v1, 10, 5);
    int64_t str2 = g10 - 1 & -16; // 0x144a
    *(int32_t *)str2 = 0x3237315f;
    *(int16_t *)(str2 + 4) = 97;
    int64_t str = function_1040(&v2); // 0x146d
    int64_t result; // rbx
    if (strcmp((char *)str, (char *)str2) != 0) {
        // 0x14bd
        result = 0;
    } else {
        // 0x148d
        int64_t v3; // bp-72
        function_1050(&v3, v1, v1);
        result = _Z6check4NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(&v3);
        function_1060(&v3);
    }
    // 0x14c2
    function_1060(&v2);
    return result;
}

// Address range: 0x12e7 - 0x13fb
// Demangled:     check4(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
int64_t _Z6check4NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(int64_t * a1) {
    int64_t v1 = (int64_t)a1; // 0x12f3
    int64_t v2; // bp-136
    function_1090(&v2, v1, 15, 5);
    int64_t str2 = g10 - 1 & -16; // 0x133f
    *(int32_t *)str2 = 0x7334316b;
    *(int16_t *)(str2 + 4) = 99;
    int64_t str = function_1040(&v2); // 0x1362
    int64_t result; // rbx
    if (strcmp((char *)str, (char *)str2) != 0) {
        // 0x13b5
        result = 0;
    } else {
        // 0x1382
        int64_t v3; // bp-104
        function_1050(&v3, v1, v1);
        result = _Z6check5NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(&v3);
        function_1060(&v3);
    }
    // 0x13ba
    function_1060(&v2);
    return result;
}

// Address range: 0x1235 - 0x12e7
// Demangled:     check5(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> >)
int64_t _Z6check5NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(int64_t * a1) {
    // 0x1235
    int64_t v1; // bp-72
    function_1090(&v1, (int64_t)a1, 20, 5);
    int64_t str2 = g10 - 1 & -16; // 0x1284
    *(int32_t *)str2 = 0x6565655f;
    *(int16_t *)(str2 + 4) = 125;
    int64_t str = function_1040(&v1); // 0x12a7
    int64_t result = strcmp((char *)str, (char *)str2) == 0;
    function_1060(&v1);
    return result;
}
```

Если внимательно посмотреть на эти функции, можно заметить, что они похожи друг на друга и состоят из двух частей:
- Вызов **function_1090** и инициализация временной строки
- Сравнение временной строки со строкой-аргументом функции
- Возврат нуля, если временная строка и входящая не совпадают (**strcmp**)

Рассмотрим первую функцию, помните мы в ассемблере заметили там кусочек "flag {" (0x00002005)?<br>
Вот он и нашелся.
```c
int64_t _Z6check1NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(int64_t * a1) {
    // ...
    function_1090(&v2, v1, 0, 5);
    int64_t v3 = _ZSteqIcSt11char_traitsIcESaIcEEbRKNSt7__cxx1112basic_stringIT_T0_T1_EEPKS5_(&v2, "flag{"); // 0x1640
    int64_t result; // rbx
    if ((char)v3 == 0) {
        // 0x1679
        result = 0;
    } else {
        // ...
        result = _Z6check2NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE(&v4);
        // ...
    }
}
```

А что за **function_1090**?
```c
// Address range: 0x1090 - 0x1096
int64_t function_1090(int64_t * a1, int64_t a2, int64_t a3, int64_t a4) {
    // 0x1090
    return _ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE6**substr**Emm();
}
```
Окончание названия вызываемой строковой функции намекает нам, что скорее всего это **substr** - <br>
взятие подстроки с i по j позиции.

Получаем, что скорее всего первая функция проверяет равенство подстроки [0, 5) значению "flag{". <br>
И если это так, возвращает результат вызова следующей строковой функции, иначе 0. <br>
Как мы помним, мы как раз не должны получить ноль в результате вызова функции проверки введенного флага.

Начало положено! :smirk_cat: 

Выпишем похожие части из остальных 4-х функций:
```с
    // _Z6check2NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
    function_1090(&v2, v1, 5, 5);
    int64_t str2 = g10 - 1 & -16; // 0x1552
    *(int32_t *)str2 = 0x66656566;
    *(int16_t *)(str2 + 4) = 97;
    int64_t str = function_1040(&v2); // 0x1575
    int64_t result; // rbx
    if (strcmp((char *)str, (char *)str2) != 0) {
        // 0x13b5
        result = 0;
    } else {
        // ...
    }
    
    // _Z6check3NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
    function_1090(&v2, v1, 10, 5);
    int64_t str2 = g10 - 1 & -16; // 0x144a
    *(int32_t *)str2 = 0x3237315f;
    *(int16_t *)(str2 + 4) = 97;
    int64_t str = function_1040(&v2); // 0x146d
    
    // _Z6check4NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
    function_1090(&v2, v1, 15, 5);
    int64_t str2 = g10 - 1 & -16; // 0x133f
    *(int32_t *)str2 = 0x7334316b;
    *(int16_t *)(str2 + 4) = 99;
    int64_t str = function_1040(&v2); // 0x1362
    
    // _Z6check5NSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEE
    function_1090(&v1, (int64_t)a1, 20, 5);
    int64_t str2 = g10 - 1 & -16; // 0x1284
    *(int32_t *)str2 = 0x6565655f;
    *(int16_t *)(str2 + 4) = 125;
```
 В каждой функции мы видим:
 - вызов **function_1090** (взятие подстроки) от очередной 5ки символов:
```c
function_1090(&v2, v1, 5, 5);
```
 - инициализацию новой строки **str2** по какому-то адресу, заполнение её значением, изменение 4-го символа (если считать с нуля):
```c
int64_t str2 = g10 - 1 & -16; // 0x1552
*(int32_t *)str2 = 0x66656566;
*(int16_t *)(str2 + 4) = 97;
```
 - инициализацию **str** от входящей строки через **function_1040**
```c
int64_t str = function_1040(&v2); // 0x1575
```
```c
// Address range: 0x1040 - 0x1046
int64_t function_1040(int64_t * a1) {
    // 0x1040
    // Скорее всего просто конструктор строки, в принципе значения не имеет
    return _ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE5c_strEv();
}
```
- сравнение **str** и **str2** на равенство и возвращения нуля или вызов следующей функции
```c
if (strcmp((char *)str, (char *)str2) != 0) {
    // 0x13b5
    result = 0;
} else {
    // ...
}
```

Отлично, теперь мы понимаем, что:
- первая функция проверяла, что первые пять символов [0, 5) введенной строки это "flag{"
- остальные функции проверяют равенство последующих пятерок символов [5, 10), [10, 15), [15, 20), [20, 25) заданным строкам

Давайте посмотрим на последнюю функцию:
```c
*(int32_t *)str2 = 0x6565655f;
*(int16_t *)(str2 + 4) = 125;
```
Видим, что по нашей логике последний символ флага - 125. Посмотрим через Python:
```python
print(chr(125)) # }
```
и это закрывающая скобка, кажется, мы на верном пути!

По адресу **str2** каждый раз кладется конкретное значение, давайте посмотрим, что это за значения:
```text
flag{
0x66656566 + 97
0x3237315f + 97
0x7334316b + 99
0x6565655f + 125
```

Структура флага ясна, декодируем с помощью Python:
```
print(chr(int('66', 16)), chr(int('65', 16)), chr(int('65', 16)), chr(int('66', 16)))  # f e e f  
print(chr(int('32', 16)), chr(int('37', 16)), chr(int('31', 16)), chr(int('5f', 16)))  # 2 7 1 _
print(chr(int('73', 16)), chr(int('34', 16)), chr(int('31', 16)), chr(int('6b', 16)))  # s 4 1 k  
print(chr(int('65', 16)), chr(int('65', 16)), chr(int('65', 16)), chr(int('5f', 16)))  # e e e _
```

И слагаемые (последние символы пятерок)
```
print(list(map(chr, [97, 97, 99, 125])))  # ['a', 'a', 'c', '}']
```

Получаем ключ: **flag{feefa271_as41kceee_}** - 25 символов, похоже на правду:smiling_imp:. <br>
Вводим в систему - неверно!

В чем может быть ошибка?...

Давайте ещё раз посмотрим на декодированные строки. <br>
Кажется, мы где-то это видели... Так в строковой же секции ассемблера!                                                                                                                             
```
0x1296	_eeef
0x1351	k14sf
0x145c	_172f
0x1564	feeff
```

Только порядок символов другой и их 5, а не 4. <br>
Ассемблеру виднее, может, мы где-то ошиблись в адресной арифметике (int16_t / int32_t / int64_t)? <br>
Видим, что последний символ у всех строк **f** - скорее всего неинициализированная область заполнилась единичками.

Но нам это и не важно, так как последний символ в программе меняется.<br>

Попробуем сохранить ассемблерный порядок символов в строках и заменить последнюю **f** на наши слагаемые.<br>
Получаем ключ - **flag{feefa_172ak14sc_eee}**
                                                                                                                                                 
Подходит! Ура!
