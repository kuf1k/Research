# Дослідження алгоритму сортування Тіма (Tim Sort)
**Сортування Тіма (Tim Sort)** -  гібридний стабільний алгоритм сортування, який поєднує в собі сортування злиттям (Merge Sort) та сортування вставкою (Insertion Sort), призначений для ефективної роботи з багатьма видами реальних даних. TimSort широко використовується у вбудованих алгоритмах сортування в мовах програмування, таких як Java, Python, C та C++. Ідея цього алгоритму полягає в сортуванні малих фрагментів за допомогою сортування вставками, а потім об'єднанні всіх великих фрагментів за допомогою функції злиття алгоритму сортування злиттям. Він був втілений в життя Тімом Пітерсом у 2002 році для використання у мові програмування Python.

**Детальніше пояснення алгоритму:**
У цьому алгоритмі масив розділяється на невеликі фрагменти. Ці фрагменти називаються `RUN`. Далі робота алгоритму  відбувається за такою логікою:

**1. Ідентифікація програлин:** Алгоритм сканує масив, щоб знайти невеликі сегменти, які вже відсортовані (`RUN-и`) . Якщо прогалина розташована у порядку спадання, вона перевертається у порядку зростання. 

**2. Сортування малих `RUN-ів` :** Якщо `RUN` коротший за фіксований розмір (зазвичай 32), він сортується за допомогою сортування вставкою, яке швидко підходить для малих або майже відсортованих даних. 

**2. Злиття`RUN-ів` :** Timsort об'єднує `RUN-и`, використовуючи правила, які забезпечують збалансоване та ефективне об'єднання, подібно до сортування злиттям, але оптимізовано для реальних даних.

## Галузі математики, що використовуються для опису алгоритму сортування Тіма (Tim Sort):
### 1. Теорія алгоритмів та математичний аналіз (Асимптотичний аналіз) : 
Математичний аналіз та теорія алгоритмів застосовуються для оцінки ефективності алгоритму, коли розмір вхідного масиву $n$ прагне до нескінченності ($n \to \infty$).

* **Нотація O-велике (O)**: визначає верхню межу часу роботи алгоритму(найгірший випадок).
* **Нотація $\Omega$-велике ($\Omega$):** задає нижню межу(найкращий випадок).
* **Нотація $\Theta$-велике ($\Theta$):** вказує на точну асимптотичну оцінку(середній випадок).

### 2. Дискретна математика (Теорія графів) : 
У дискретній математиці процес роботи Timsort моделюється не просто класичним деревом рекурсії, а за допомогою орієнтованого графу станів стеку та лінійних зв'язаних структур (набору `RUN-ів`).

### 3. Математична логіка: 
Оскільки Timsort є складним комбінованим алгоритмом (використовує цикли для пошуку `RUN-ів`, сортування вставками для малих `RUN-ів` та стек для їхнього злиття), суворе логічне доведення його коректності поєднує метод математичної індукції та метод формальних інваріантів.


## Реалізація алгоритму сортування Тіма (Tim Sort):
### Реалізація за допомогою C++ : 
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

const int minRUN = 32;

int calcMinRun(int n) {
    int r = 0;
    while (n >= minRUN) {
        r |= (n & 1);
        n >>= 1;
    }
    return n + r;
}

void insertionSort(vector<int>& arr, int left, int right) {
    for (int i = left + 1; i <= right; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= left && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

void merge(vector<int>& arr, int l, int m, int r) {
    vector<int> left(arr.begin() + l, arr.begin() + m + 1);
    vector<int> right(arr.begin() + m + 1, arr.begin() + r + 1);

    int i = 0, j = 0, k = l;
    while (i < left.size() && j < right.size()) {
        if (left[i] <= right[j]) arr[k++] = left[i++];
        else arr[k++] = right[j++];
    }
    while (i < left.size()) arr[k++] = left[i++];
    while (j < right.size()) arr[k++] = right[j++];
}


int findRun(vector<int>& arr, int start, int n) {
    int end = start + 1;
    if (end == n) return end;

    if (arr[end] < arr[start]) {

        while (end < n && arr[end] < arr[end - 1]) end++;
        reverse(arr.begin() + start, arr.begin() + end);
    } else {

        while (end < n && arr[end] >= arr[end - 1]) end++;
    }
    return end;
}


void timsort(vector<int>& arr) {
    int n = arr.size();
    int minRun = calcMinRun(n);
    vector<pair<int,int>> runs;

    int i = 0;
    while (i < n) {
        int runEnd = findRun(arr, i, n);
        int runLen = runEnd - i;

        if (runLen < minRun) {
            int end = min(i + minRun, n);
            insertionSort(arr, i, end - 1);
            runEnd = end;
        }
        runs.push_back({i, runEnd});
        i = runEnd;

        while (runs.size() > 1) {
            int l1 = runs[runs.size() - 2].first;
            int r1 = runs[runs.size() - 2].second;
            int l2 = runs[runs.size() - 1].first;
            int r2 = runs[runs.size() - 1].second;

            int len1 = r1 - l1;
            int len2 = r2 - l2;
            if (len1 <= len2) {
                merge(arr, l1, r1 - 1, r2 - 1);
                runs.pop_back();
                runs[runs.size() - 1] = {l1, r2};
            } else break;
        }
    }

    while (runs.size() > 1) {
        int l1 = runs[runs.size() - 2].first;
        int r1 = runs[runs.size() - 2].second;
        int l2 = runs[runs.size() - 1].first;
        int r2 = runs[runs.size() - 1].second;
        merge(arr, l1, r1 - 1, r2 - 1);
        runs.pop_back();
        runs[runs.size() - 1] = {l1, r2};
    }
}

int main() {
    vector<int> arr = {5, 21, 7, 23, 19, 10, 1, 3, 2};
    timsort(arr);
    for (int x : arr) cout << x << " ";
    cout << endl;
}
```

### Вхідні дані, які обробляє алгоритм :

  1. Динамічний масив цілих чисел (вектор) (std::vector<int>& arr).
  2. Індекси та межі підмасивів (int left, int right, int start, int l, int m, int r): Цілі числа, що визначають точні межі `RUN-ів `, які обробляються на поточній ітерації циклів.
* **Початковий стан:**  `std::vector<int> arr = {5, 21, 7, 23, 19, 10, 1, 3, 2} ` (Елементи розташовані в недетермінованому (несортованому) порядку.
### Результат роботи алгоритму:
<img width="354" height="96" alt="image" src="https://github.com/user-attachments/assets/8359c4be-3305-45f7-aaa2-1e05bc57b7a6" />

Програма завершилась успішно!

* : Отримуємо оновлений динамічний масив  `std::vector<int>& arr` під тим самим ідентифікатором у пам'яті ( оскільки сортування відбувається на місці (In-place)).
* **Кінцевий стан :** Усі 9 елементів масиву повністю відсортовані в порядку зростання. Після завершення роботи алгоритму timsort, виконання ітераційного циклу та виведення в консол, ми отримуємо чітку послідовність:
`1 2 3 5 7 10 19 21 23`

 ### Додаткові обмеження та властивості 
 * **Часова складність:**  Timsort є адаптивним алгоритмом сортування, що означає, що його ефективність динамічно підлаштовується під початковий стан даних. Він проектувався як гібрид, здатний поєднати найкращі швидкісні характеристики обох своїх «батьків» (Merge Sort та Insertion Sort).

    * **Найкращий випадок ( $\Omega(n)$ ):** Якщо вхідний масив уже повністю або майже відсортований, Timsort демонструє надзвичайну ефективність, повністю повторюючи поведінку Insertion Sort, який на впорядкованих даних працює за лінійний час. Алгоритм робить лише один лінійний прохід по масиву, виявляє, що дані складаються з готових `RUN-ів`, і завершує роботу. Це робить його кардинально швидшим за Merge Sort, який у цьому випадку працював би набагато довше.

    * **Середній випадок ( $\Theta(n \log n)$ ):** На частково впорядкованих або випадкових даних Timsort ефективно знаходить наявні монотонні послідовності (`RUN-и`), штучно розширює надто малі фрагменти за допомогою логіки Insertion Sort, а потім зливає їх за логікою Merge Sort. Завдяки такому гібридному підходу, а також використанню режиму «галопу» (Galloping Mode), середній час роботи Timsort на реальних даних є суттєво меншим, ніж у класичного Merge Sort, і в рази меншим за Insertion Sort.

    * **Найгірший випадок ( $\Theta(n \log n)$$ ):** Якщо масив є абсолютно хаотичним і не містить жодних готових послідовностей, Timsort розбиває його на мінімальні блоки розміром `minrun`, сортує їх вставкою, а потім ітеративно зливає разом. Верхня межа Timsort ніколи не перевищить лінійно-логарифмічну залежність $O(n \log n)$. Цим він повністю копіює надійність Merge Sort і демонструє колосальну перевагу над Insertion Sort.

 
* **Просторова складність:** Timsort розроблений так, щоб мінімізувати витрати пам'яті, які є головним мінусом Merge Sort, але при цьому забезпечити масштабованість, недоступну для Insertion Sort. Його просторова складність становить $O(n)$


* **Стабільність :** Timsort є стабільним, це досягається завдяки тому, що обидва базові алгоритми в його основі також є стабільними.

* **Ефективність для зв'язаних списків :** Хоча для векторів та масивів алгоритм потребує $O(n)$ додаткової пам'яті, при сортуванні структур типу зв'язаний список сортування злиттям (Merge Sort) може працювати за $O(1)$ додаткової пам'яті. Це відбувається, через те, що у списках елементи не лежать у пам'яті послідовно, а зв'язані вказівниками і процедуру злиття можна виконати, просто змінюючи напрямок вказівників, без створення тимчасових копій елементів. Тобто для списків Merge Sort стає In-place алгоритмом, зберігаючи часову складність $O(n \log n)$. 

* **Режим «галопу» (Galloping Mode):** Це унікальна властивість Timsort, якої немає ні в Merge Sort, ні в Insertion Sort. Коли алгоритм зливає два великих `RUN-и`, він збирає статистику порівнянь. Якщо елементи одного підмасиву виявляються меншими за поточний елемент іншого багато разів поспіль, Timsort замість поштучного порівняння за $O(1)$ перемикається на бінарний пошук. Це дозволяє миттєво знаходити місце для вставки цілих кластерів даних за логарифмічний час $O(\log k)$. У цьому аспекті Timsort кардинально обходить як Merge Sort та Tim Sort.

## Практичні вимірювання ефективності алгоритму сортування Тіма (Tim Sort)
Для експериментального підтвердження теоретичних оцінок складності алгоритму було використано наведений програмний код на мові C++. Тестування проводилося на базі динамічного контейнера (вектора) розміром $N = 100,000$ елементів типу `int`. Заміри часу виконання здійснювалися за допомогою високоточного системного таймера з бібліотеки `<chrono>` (`high_resolution_clock`).
### 1. Методика проведення експерименту
Ефективність алгоритму вимірювалася для трьох принципово різних типів початкових даних:
1. **Випадковий масив(Середній випадок):**  Елементи генерувалися хаотично у діапазоні від `0` до `99999` за допомогою функції `rand()`. Це відповідає середньому випадку часової складності $\Theta(n \log n)$.
2. **Впорядкований масив (Найкращий випадок):** Масив заповнювався числами за зростанням (від `0` до `99999`). Це дозволяє перевірити унікальну лінійну складність алгоритму $\Omega(n)$ та його адаптивність до вже готових даних.
3. **Зворотний масив (Найгірший випадок):**  Масив заповнювався числами у спадному порядку (від `100000` до `1`). Це дозволяє перевірити, як функція `findRun` ефективно виявляє спадні послідовності та розгортає їх за лінійний час $O(k)$, оптимізуючи підсумкову складність до $O(n \log n)$.

#### Релізація експерименту за допомогою C++
```cpp
#include <iostream>
#include <vector>
#include <algorithm>
#include <ctime>
#include <chrono>
#include <fstream>

using namespace std;
using namespace std::chrono;

const int minRUN = 32;

int calcMinRun(int n) {
    int r = 0;
    while (n >= minRUN) {
        r |= (n & 1);
        n >>= 1;
    }
    return n + r;
}

void insertionSort(vector<int>& arr, int left, int right) {
    for (int i = left + 1; i <= right; i++) {
        int key = arr[i];
        int j = i - 1;
        while (j >= left && arr[j] > key) {
            arr[j + 1] = arr[j];
            j--;
        }
        arr[j + 1] = key;
    }
}

void merge(vector<int>& arr, int l, int m, int r) {
    vector<int> left(arr.begin() + l, arr.begin() + m + 1);
    vector<int> right(arr.begin() + m + 1, arr.begin() + r + 1);

    int i = 0, j = 0, k = l;
    while (i < left.size() && j < right.size()) {
        if (left[i] <= right[j]) arr[k++] = left[i++];
        else arr[k++] = right[j++];
    }
    while (i < left.size()) arr[k++] = left[i++];
    while (j < right.size()) arr[k++] = right[j++];
}

int findRun(vector<int>& arr, int start, int n) {
    int end = start + 1;
    if (end == n) return end;

    if (arr[end] < arr[start]) {
        while (end < n && arr[end] < arr[end - 1]) end++;
        reverse(arr.begin() + start, arr.begin() + end);
    } else {
        while (end < n && arr[end] >= arr[end - 1]) end++;
    }
    return end;
}

void timsort(vector<int>& arr) {
    int n = arr.size();
    int minRun = calcMinRun(n);
    vector<pair<int,int>> runs;

    int i = 0;
    while (i < n) {
        int runEnd = findRun(arr, i, n);
        int runLen = runEnd - i;

        if (runLen < minRun) {
            int end = min(i + minRun, n);
            insertionSort(arr, i, end - 1);
            runEnd = end;
        }
        runs.push_back({i, runEnd});
        i = runEnd;

        while (runs.size() > 1) {
            int l1 = runs[runs.size() - 2].first;
            int r1 = runs[runs.size() - 2].second;
            int l2 = runs[runs.size() - 1].first;
            int r2 = runs[runs.size() - 1].second;

            int len1 = r1 - l1;
            int len2 = r2 - l2;
            if (len1 <= len2) {
                merge(arr, l1, r1 - 1, r2 - 1);
                runs.pop_back();
                runs[runs.size() - 1] = {l1, r2};
            } else break;
        }
    }

    while (runs.size() > 1) {
        int l1 = runs[runs.size() - 2].first;
        int r1 = runs[runs.size() - 2].second;
        int l2 = runs[runs.size() - 1].first;
        int r2 = runs[runs.size() - 1].second;
        merge(arr, l1, r1 - 1, r2 - 1);
        runs.pop_back();
        runs[runs.size() - 1] = {l1, r2};
    }
}

int main() {
    const int ARRAY_SIZE = 100000;
    srand(time(0));

    // 1. ВИПАДКОВИЙ МАСИВ (Середній випадок)
    vector<int> randomArray(ARRAY_SIZE);
    for (int i = 0; i < ARRAY_SIZE; ++i) {
        randomArray[i] = rand() % 100000;
    }

    auto startRandom = high_resolution_clock::now();
    timsort(randomArray);
    auto stopRandom = high_resolution_clock::now();
    auto durationRandom = duration_cast<microseconds>(stopRandom - startRandom);


    // 2. ВЖЕ ВПОРЯДКОВАНИЙ МАСИВ (Найкращий випадок)
    vector<int> sortedArray(ARRAY_SIZE);
    for (int i = 0; i < ARRAY_SIZE; ++i) {
        sortedArray[i] = i;
    }

    auto startSorted = high_resolution_clock::now();
    timsort(sortedArray);
    auto stopSorted = high_resolution_clock::now();
    auto durationSorted = duration_cast<microseconds>(stopSorted - startSorted);


    // 3. ЗВОРОТНИЙ МАСИВ (Найгірший випадок теорії)
    vector<int> reversedArray(ARRAY_SIZE);
    for (int i = 0; i < ARRAY_SIZE; ++i) {
        reversedArray[i] = ARRAY_SIZE - i;
    }

    auto startReversed = high_resolution_clock::now();
    timsort(reversedArray);
    auto stopReversed = high_resolution_clock::now();
    auto durationReversed = duration_cast<microseconds>(stopReversed - startReversed);


    // === ЗАПИС РЕЗУЛЬТАТІВ У CSV ФАЙЛ ===
    ofstream csvFile("timsort_results.csv");
    if (csvFile.is_open()) {
        csvFile << "Array_Size,Best_Case_Mks,Average_Case_Mks,Worst_Case_Mks\n";
        csvFile << ARRAY_SIZE << ","
                << durationSorted.count() << ","
                << durationRandom.count() << ","
                << durationReversed.count() << "\n";
        csvFile.close();
    }


    cout << "=== Timsort Test Results (N = " << ARRAY_SIZE << ") ===" << endl << endl;

    cout << "1. Random Array (Average Case):" << endl;
    cout << "   " << durationRandom.count() << " microseconds "
         << "(" << durationRandom.count() / 1000.0 << " milliseconds)" << endl << endl;

    cout << "2. Already Sorted Array (Best Case):" << endl;
    cout << "   " << durationSorted.count() << " microseconds "
         << "(" << durationSorted.count() / 1000.0 << " milliseconds)" << endl << endl;

    cout << "3. Reversed Array (Worst Case/Optimized Run):" << endl;
    cout << "   " << durationReversed.count() << " microseconds "
         << "(" << durationReversed.count() / 1000.0 << " milliseconds)" << endl << endl;

    return 0;
}
```

### 2. Результати практичних замірів 

[timsort_results.csv](https://github.com/user-attachments/files/29294368/timsort_results.csv)
```csv
Array_Size,Best_Case_Mks,Average_Case_Mks,Worst_Case_Mks
100000,195,24537,755
```

| Тип вхідного масиву | Час виконання (мікросекунди) | Час виконання (мілісекунди) | Big O |
| :--- | :---: | :---: | :---: |
| **Випадковий** | `24537` мкс | `24.537` мс | Середній/Найгірший випадок $\Theta(n \log n)$ |
| **Впорядкований** | `195` мкс | `0.195` мс | Найкращий випадок $\Omega(n)$ |
| **Зворотний** | `755` мкс | `0.755` мс | Оптимізований випадок $O(n)$ |

<img width="556" height="159" alt="image" src="https://github.com/user-attachments/assets/611dd47f-8139-4c77-98e5-e1d2d1fde863" />
<img width="368" height="202" alt="image" src="https://github.com/user-attachments/assets/a3fd3da0-e51b-4050-9375-76e685193b68" />

### 3. Аналіз отриманих результатів

1. **Підтвердження адаптивності алгоритму:** Практичні заміри показали колосальну різницю між часом обробки випадкового масиву (`24537 мкс`) та впорядкованих послідовностей (`195 мкс` та `755 мкс`). Це експериментально підтверджує, що Timsort є високадаптивним гібридним алгоритмом. На відміну від класичного сортування злиттям (Merge Sort), де структура розбиття є строго детермінованою і завжди виконує фіксований обсяг роботи за $\Theta(n \log n)$, Timsort миттєво підлаштовується під початковий стан даних, перемикаючись на лінійну складність $\Omega(n)$, якщо масив повністю або частково впорядкований.
3. **Аналіз коливань часу:** Результати замірів наочно демонструють одну з головних  фішок Timsort - роботу функції findRun:

   * Для *впорядкованого масиву* час склав усього `195 мкс`, оскільки алгоритм за один лінійний прохід зрозумів, що дані вже готові.
   
   * Для зворотного масиву час склав усього `755 мкс`, що в десятки разів швидше за випадковий масив. В  Timsort функція `findRun` вчасно ідентифікувала спадний рух елементів і розгорнула всю послідовність за один лінійний прохід `std::reverse`. Натомість випадковий масив виявився найважчим випадком (`24.537 мс`), оскільки доводилося постійно викликати внутрішній `insertionSort` для формування базових `RUN-ів` розміром 32 елементи, а потім масовано об'єднувати їх у стеку через `merge`.

5. **Висновок для практичного використання (Timsort vs Merge Sort vs Insertion Sort):**  Для великого масиву розміром $100,000$ елементів Timsort демонструє ідеальний баланс швидкості та прагматизму. Він повністю увібрав у себе здатність сортування вставкою (Insertion Sort)  за мікросекунди, обробляти вже готові чи реверсні дані (чого не вміє робити Merge Sort). Водночас на повністю хаотичних масивах Timsort страхує систему від квадратичного падіння продуктивності, притаманного сортуванню вставкою (Insertion Sort), та гарантує складність $O(n \log n)$, як у сортування злиттям (Merge Sort). Це робить Timsort одним із найефективніших алгоритмів для сортування реальних даних (Real-world data), які практично ніколи не бувають абсолютно випадковими.

*Питання : Тоді коли саме ефективно використовувати сортування вставкою(Insertion Sort), сортування злиттям(Merge Sort) та сортування Тіма (Tim Sort)??* 

Відповідь :
* **Для малих або вже частково відсортованих масивів:** Найефективнішим є сортування вставками (Insertion Sort) через відсутність додаткових витрат пам'яті.
* **Для великих і повністю хаотичних даних:** Merge Sort забезпечує швидку та детерміновану обробку, проте вимагає суттєвих $O(n)$ витрат оперативної пам'яті.
* **Для універсального застосування на практиці:** Timsort є найкращим рішенням, оскільки він економить ресурси пам'яті, неймовірно добре утилізує природну впорядкованість даних і гарантує високу швидкість $O(n \log n)$ у найгіршому сценарії. Саме тому Timsort заслужено став стандартом у сучасних мовах програмування.

## Реальні приклади використання алгоритму сортування Тіма (Tim Sort): 

1. Використовується як алгоритм сортування за замовчуванням у Python (`sorted()`, `list.sort()`) та Java (починаючи з Java 7 для `Arrays.sort()` на об'єктах).
2. Добре працює в системах баз даних та пошукових системах, де важливе значення має дотримання порядку рівних ключів.
3. Використовується в Android, Swift та інших бібліотеках завдяки своїй стабільності та ефективності.

## Деякі дискусії, пов'язані із алгоритмом сортування злиттям (Merge Sort):
1. Обговорення корисних аспектів сортування злиттям ( https://www.quora.com/What-is-merge-sort-and-how-is-it-useful )
2. Обговорення можливостей покращення сортування злиттям ( https://www.quora.com/How-can-we-improve-the-performance-of-a-merge-sort ) 
3. Обговорення стабільності сортування злитям ( https://www.quora.com/Is-merge-sort-a-stable-sorting-algorithm )


## Використані джерела: 



 

