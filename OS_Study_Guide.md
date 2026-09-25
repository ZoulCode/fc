# دليل مذاكرة CIS353 – Operating Systems (نحو A+)
### مبني بالكامل على محتوى الملف اللي بعتّه (المحاضرات، اللابات، الـ Sheets، الكويزات الحقيقية، والميدل تيرم بحلوله النموذجية)

---

## 0) الصورة الكبيرة (اقرأها الأول)

المادة عندك مقسّمة لجزئين واضحين، ولازم تفرّق بينهم وانت مذاكر:

| الجزء | المحتوى | هيتحاسب إزاي |
|---|---|---|
| **Lectures (10)** | نظرية أنظمة التشغيل: OS overview, Processes, Virtual Memory (HW+OS role), Uni-CPU Scheduling, Concurrency, Deadlocks, Multi-CPU Scheduling | Midterm + Final (نظري + مسائل حسابية) |
| **Labs (1,2,3,4,6,7,9)** | تطبيق عملي: بناء نظام تشغيل بسيط (FOS) بلغة C لإدارة الذاكرة (Labs 1-6)، ثم Multithreading بلغة C# (.NET) لحل مشاكل التزامن (Labs 7,9) | Hands-on exam + Project + Lab bonuses |

**ملحوظة مهمة:** مفيش Lab 5 و Lab 8 في الملف اللي بعتّه (يبدو إنهم كانوا لابات مراجعة/تطبيق مباشر مش موزعة كملفات منفصلة، أو اتدمجوا). ركّز على اللي موجود.

**شكل الامتحان الحقيقي (من الـ Midterm Model Answer اللي في ملفاتك):**
- **جزء أول [6 درجات] "General Concepts":** جدول Process vs Thread (صح/غلط لكل خاصية)، وتكملة دورة الـ System Call (رسمة بترقيم الخطوات)، وأسئلة على أدوار الـ OS (Illusionist/Governor/History Teacher) وعلى الـ 7-state diagram.
- **جزء تاني [9(+1) درجات] "Memory":**
  - Segmentation (Worst Fit + Compaction) – ابنِ segment table.
  - HW Role – حسابات Multilevel Paging (عدد المستويات، عدد البتّات، الـ internal fragmentation).
  - OS Role – Trace لخوارزمية استبدال صفحات (LRU/Clock) واحسب page faults, disk access, والمحتوى النهائي لل resident set.
- **الفاينل** غالبًا هياخد نفس الشكل بس على باقي المنهج: Scheduling (Uni & Multi-CPU) + Concurrency (Semaphores/Monitors) + Deadlocks.

**الدرس الأهم من كل الكويزات اللي راجعتها:** الأسئلة **مش حفظ**. هي نفس "الشكل" بيتكرر بأرقام مختلفة (نفس مسألة الـ multilevel paging، نفس trace الـ LRU، نفس جدول الـ scheduling) — يعني لو فهمت الطريقة مش الرقم، هتحل أي نسخة يجيلك.

---

## 1) Lecture 1 – OS Overview

### قبل ما تدخل المحاضرة/تتعامل مع أي سؤال، لازم تكون عارف:
- **إيه هو الـ OS:** البرنامج اللي بيدير التنافس (competition) بين البرامج على الموارد (CPU, Memory, I/O Devices).
- **3 أدوار أساسية للـ OS** (دي بالحرف جت في الميدل والـ retake):
  1. **Illusionist:** بيدّي كل process وهم إن عنده جهاز كامل لوحده (virtual space لا نهائي، ملفات، threads...) بينما الحقيقة إن الموارد مشتركة.
  2. **Governor:** بيدير الموارد فعليًا (allocation, protection, isolation, context switch) — وده بييجي بـ"ضريبة" (Tax = overhead زي وقت الـ context switch وفقدان الـ cache).
  3. **History Teacher:** بيتعلم من تصميمات قديمة عشان يتوقع احتياجات المستقبل (تصميم الـ OS بيتغير مع تطور الـ hardware).
- **System Call Cycle** (جالت في الميدل كرسمة تكميل خطوات) — رتّبها كده:
  1. البرنامج بينده syscall ويمرر الـ parameters (غالبًا عبر CPU registers).
  2. الـ Hardware يحفظ الـ flags/return address/old stack، ويحول من User Mode لـ Kernel Mode.
  3. الـ Hardware يبدّل لـ Kernel Stack.
  4. الـ OS (handler) يحفظ حالة الـ caller (الـ CPU registers).
  5. الـ OS يشوف رقم الـ syscall في جدول (زي Interrupt Descriptor Table) وينده الـ function المناسبة.
  6. لما تخلص، يرجع من Kernel Mode لـ User Mode.
- **الفرق بين Process و Thread** (جدول اتسأل فيه بالحرف):

| الخاصية | Process | Thread |
|---|---|---|
| Switching overhead | عالي | منخفض |
| سهولة التواصل (communication) | صعب (memory منفصلة) | سهل (memory مشتركة) |
| Sharing overhead | عالي | منخفض |
| مستوى الـ Protection | عالي (معزول) | منخفض (مشترك مع باقي threads نفس الـ process) |
| بيتكون من CPU registers + private stack | (كل thread كده) | ✔ |
| ممكن يتوازى على أكتر من CPU | ✔ | ✔ |

- **متى يُعتبر السطر محتاج System Call؟** (سؤال جالك في الكويز): أي عملية بتحتاج تدخل على الـ Kernel أو موارد الجهاز — زي: توليد رقم عشوائي حقيقي (`Random`)، تخصيص مساحة ذاكرة جديدة (`new int[N]`)، أو الكتابة على الشاشة (`Console.WriteLine`). أما العمليات الحسابية الداخلية (loop عادي، جمع أرقام) **مش** محتاجة syscall.

### أسئلة متوقعة (نفس نمط اللي جالك في الكويزات فعلاً):
- "أي من التالي **مش** من أهداف الـ OS؟" (تحقق واحدة بس مش من ضمنها — زي "Check main I/O devices" مثلاً مش هدف مباشر).
- "إيه الهدف من الـ Multiprogramming؟" → **زيادة استغلال المعالج (increase CPU utilization)**.
- كود فيه أسطر مرقّمة، واختار أي الأسطر تستدعي System Call.
- ربط كل جملة (زي "بيديدي كل process فضاء افتراضي" أو "بيسمح بالتبديل بين العمليات") بالدور المناسب (Illusionist/Governor/History Teacher).

---

## 2) Lecture 2 – Process, Switch & Locks

### المفاهيم الأساسية:
- **الـ 7-State Process Model** (ده أهم حاجة في المحاضرة وجه بالحرف في الـ Retake Midterm):
  `New → Ready → Running → Blocked → Exit`، بالإضافة لحالتين للـ Suspend:
  - **Ready/Suspend:** العملية في الـ disk، جاهزة للتنفيذ بس لسه مش محمّلة في الـ RAM.
  - **Blocked/Suspend:** العملية في الـ disk وكمان مستنية حدث معيّن (I/O مثلاً).
  
  احفظ *سبب* كل انتقال، مش بس اسم الحالة:
  - `Ready → Ready/Suspend`: مفيش مكان في الـ RAM (محتاج نـ swap-out عملية عشان نعمل مكان).
  - `Blocked → Blocked/Suspend`: نفس السبب، لكن العملية أصلاً كانت Blocked.
  - `Blocked/Suspend → Ready/Suspend`: الحدث اللي كانت مستنياه حصل.
  - `Ready/Suspend → Ready`: لما مفيش عمليات Ready تانية في الـ RAM (أو أولوية العملية المعلّقة أعلى).
  - `New → Ready`: العملية اتحمّلت في الذاكرة.
  - `Running → Exit`: خلصت أو اتقفلت (aborted).
  - `Running → Ready`: انتهت الـ time slice بتاعتها (preempted).
  - `Running → Blocked`: محتاجة حدث (I/O أو page fault).
- **Context Switch:** حفظ حالة الـ process الحالية (CPU registers) وتحميل حالة العملية الجاية. له تكلفة (Tax): وقت الحفظ/التحميل + فقدان محتوى الـ Cache.
- **Spinlocks / Locks:** الفرق بينهم وبين الـ semaphores هيتوضح أكتر في محاضرة 7-8، بس المفاهيم الأساسية (acquire/release, busy-waiting) بتتقدم هنا.

### أسئلة متوقعة:
- تكملة الـ 7-state diagram بأسباب الانتقال (زي الميدل بالظبط).
- "إيه حالة الـ Process في وقت X؟" (Ready/Running/Blocked...) بناءً على Gantt chart.
- تتبّع كود فيه أكتر من Process بتستخدم Spinlocks (زي في الكويزات: `acquire(U)`, `acquire(V)`...) واحسب أقصى/أقل عدد مرات ممكن تطبع فيها كل عملية حرف معيّن.

---

## 3) Lectures 3, 4, 5 – Virtual Memory (HW Role I & II, OS Role)

### المفاهيم الأساسية (لازم تبقى جاهزة قبل أي محاضرة فيهم):

**أ) Address Translation:**
- Virtual Address = Page Number + Offset.
- Offset bits = log2(page size).
- Page Number bits = log2(virtual memory size / page size).

**ب) Multilevel Paging (أهم نوع حسابات في الامتحان – جالت في الميدل والـ Retake بالحرف):**
- كل مستوى (level) بياخد عدد بتّات = log2(عدد الـ entries اللي ممكن تتخزن في صفحة واحدة).
  - عدد الـ entries في الصفحة الواحدة = Page Size / حجم كل Entry.
- **أقصى عدد بتّات للـ index في أي مستوى** = log2(Page Size / Entry Size).
- **أقل عدد مستويات (Min # levels)** = (Page Number bits) ÷ (أقصى بتّات لكل مستوى) — لو مش قسمة مظبوطة، اعمل round up.
- **Internal Fragmentation في جداول الصفحات:** بتحصل لما آخر مستوى ميتملاش بالكامل (يعني عدد الـ entries المطلوبة أقل من سعة الصفحة). لاحظ من الكويزات: **تقليل حجم الـ Virtual Memory** هو اللي بيسبب Internal Fragmentation (لأن آخر مستوى هيبقى فاضي جزئيًا)، مش زيادته. اتأكد تفهم *ليه* مش تحفظ بس.

**ج) TLB (Translation Lookaside Buffer):**
- Cache صغير للـ (Page#, Frame#) عشان تسرّع الترجمة وتقلل عدد مرات الوصول للـ RAM.
- لو الصفحة موجودة في TLB (TLB hit) → توصل للـ Frame على طول من غير ما تدخل جداول الصفحات في الـ RAM.
- لو مش موجودة (TLB miss) → لازم تدخل جداول الصفحات (كل مستوى = access واحدة للـ RAM).

**د) Demand Paging / Pre-paging / Resident Set:**
- Demand Paging: الصفحة متتحمّلش إلا لما تتطلب فعلًا (أول access ليها = page fault إجباري "Cold fault").
- Load-time Pre-paging: صفحات معيّنة بتتحمّل من الأول (مش بانتظار أول access).
- Resident Set Size: أقصى عدد صفحات ممكن تكون موجودة في الـ RAM لنفس الـ process في نفس الوقت.

**هـ) خوارزميات استبدال الصفحات (Page Replacement) – أهم جزء حسابي:**
| الخوارزمية | الفكرة |
|---|---|
| **FIFO** | تستبدل أقدم صفحة دخلت الذاكرة |
| **LRU (Least Recently Used)** | تستبدل الصفحة اللي "أطول مدة" من غير استخدام |
| **OPT (Optimal)** | تستبدل الصفحة اللي هتُستخدم "أبعد وقت" في المستقبل (نظري، مش قابل للتطبيق فعليًا، بس بيتقاس بيه أداء باقي الخوارزميات) |
| **Clock (Second Chance)** | تستخدم "use bit"، بتدور على الصفحات زي عقارب الساعة وتدّي فرصة تانية للصفحة لو use bit=1 |
| **Working Set** | تحتفظ فقط بالصفحات اللي اتستخدمت في آخر Δ (window) من الزمن |

**كل مسألة بتديك:**
1. سلسلة page references (r/w).
2. حجم الصفحة، حجم الـ resident set، صفحات pre-paged (لو موجودة).
3. الخوارزمية.

**وبتسألك عن:**
- عدد الـ page faults (فرّق بين cold fault "أول ظهور" و warm fault "استبدال").
- عدد مرات القراءة/الكتابة على الـ Disk:
  - Disk Read = عدد الـ page faults (لازم نجيب الصفحة من الـ disk).
  - Disk Write = بس لو الصفحة اللي هتتشال كانت "متعدّلة" (dirty/modified bit = 1، يعني حصلها Write قبل كده).
- محتوى الـ resident set في الآخر.

**نصيحة:** اعمل جدول زمني (timeline) لكل reference وسجّل فيه: هل fault ولا لأ، مين اتشال (لو استبدال)، وهل اللي اتشال كان معدّل (dirty) ولا لأ. ده هيخليك ماتلخبطش.

**و) Segmentation:**
- كل Segment ليه Base + Limit.
- استراتيجيات التخصيص: First Fit, Best Fit, Worst Fit.
- **Compaction:** تجميع كل الأجزاء الحرة في مكان واحد (بيتفعل لو فيه Fragmentation والـ OS مسموحله يعمل Compaction).
- Segment Sharing: لو process تاني عايز يستخدم نفس الـ Segment (زي في الميدل)، ياخد نفس الـ Base والـ Limit.

### أسئلة متوقعة (من الشيتات + الميدل + الكويزات الحقيقية):
- "لو الـ Virtual Memory اتزودت لـ X، احسب: أقصى بتّات للـ index، أقل عدد مستويات، مقدار الـ internal fragmentation."
- Trace كامل لـ LRU/Clock/FIFO مع resident set معين واحسب: faults, disk reads, disk writes, المحتوى النهائي.
- بناء Segment Table لعملية جديدة (باستخدام Worst/Best/First Fit + هل يحتاج Compaction ولا لأ).
- "هل تغيير سياسة الاستبدال بيأثر على عدد مرات الوصول للـ Disk؟" — الإجابة بتتوقف على تفاصيل المسألة، ففهم المنطق أهم من حفظ إجابة عامة.

---

## 4) Lecture 6 – Uni-Processor Scheduling

### الخوارزميات (لازم تعرف تعمل Trace/Gantt Chart لكل واحدة):

| الخوارزمية | Preemptive؟ | الفكرة |
|---|---|---|
| **FCFS** | لا | أول واحد يوصل، أول واحد يتنفذ |
| **RR (Round Robin)** | نعم | كل process ياخد Time Quantum ثابت بالدور |
| **SPN (Shortest Process Next)** | لا | يختار أقصر Service Time بين الـ Ready |
| **SRT (Shortest Remaining Time)** | نعم | زي SPN بس بيقارن بالـ remaining time وممكن يقاطع لو وصلت عملية أقصر |
| **HRRN (Highest Response Ratio Next)** | لا | Response Ratio = (Wait Time + Service Time) / Service Time — يختار الأعلى نسبة (بيحل مشكلة الـ starvation في SPN) |
| **MLFQ (Multilevel Feedback Queue)** | نعم | مستويات كل واحد له Quantum مختلف؛ العملية اللي ماتخلصش في مستوى بتنزل للمستوى الأقل أولوية |

### المقاييس اللي المفروض تحسبها لكل عملية:
- **Finish Time**
- **Turnaround Time** = Finish − Arrival
- **Normalized Turnaround** = Turnaround / Service Time
- **Wait Time** = Turnaround − Service Time

### طريقة الحل خطوة بخطوة (زي الشيت بالحرف):
1. ارسم Execution Pattern/Trace: في كل لحظة زمن، مين اللي واصل، مين في الـ Ready Queue، مين شغال على الـ CPU.
2. لما تعمل RR أو MLFQ، لازم ترسم حركة الـ Ready Queue خطوة بخطوة (مين بيدخل ومين بيخرج) — ده أكتر جزء بيغلط فيه الطلبة.
3. احسب الجدول في الآخر.

### أمثلة أسئلة حقيقية جتلك في الكويزات:
- Gantt chart لعمليتين/تلاتة بيهم CPU/I/O bursts وTime Now معين، واسأل: "إيه finish time بتاع Process X؟" أو "إيه حالة Process Y في وقت 15؟"
- SPN: ترتيب تنفيذ 5 processes بأزمنة وصول وخدمة مختلفة.
- MLFQ: تصميم الـ Quantum لكل مستوى عشان تحقق نسبة معيّنة من العمليات تخلص في كل مستوى (زي 40%-40%-20%).
- HRRN: حساب الـ Response Ratio في كل لحظة اختيار.

---

## 5) Lectures 7 & 8 – Concurrency

### المفاهيم الأساسية:
- **Race Condition:** لما أكتر من process/thread بيوصلوا لنفس الـ data في نفس الوقت والنتيجة بتتغير حسب ترتيب التنفيذ.
- **Critical Section:** الجزء من الكود اللي بيوصل لموارد مشتركة.
- **Mutual Exclusion:** لو process داخل الـ Critical Section، محدش غيره يدخل في نفس الوقت.
- **3 شروط للحل الصحيح:** Mutual Exclusion + Progress (مفيش تأجيل غير ضروري) + Bounded Waiting (مفيش starvation).

### Semaphore:
- بيتكون من: **Value** + **Queue of blocked processes**.
- عمليتين ذريّتين (atomic): **wait(S)** (تنقص القيمة، لو سالبة تستنى) و **signal(S)** (تزود القيمة، لو فيه مستنيين توقظ واحد).
- Binary Semaphore (0 أو 1) = بيشتغل زي Lock للـ Mutual Exclusion.
- Counting Semaphore = بيتحكم في عدد الوصول لموارد متعددة (زي الكراسي، الـ dependency بين Producer/Consumer).

### نقطة مهمة جدًا اتكررت في أكتر من كويز (فرّق بينهم كويس):
| الحالة | لو حصل `signal` قبل `wait` غلط | لو حصل `release` قبل `acquire` (Sleep Lock) |
|---|---|---|
| النتيجة | ممكن أكتر من process يدخلوا الـ Critical Section في نفس الوقت (مش deadlock، لكن انتهاك للـ Mutual Exclusion) | ممكن يحصل **Deadlock** (لأن الـ Lock مش بيسمح بالدخول تاني من غير الترتيب الصحيح) |

يعني: **Semaphore غلط الترتيب فيه = خرق للـ mutual exclusion. Lock غلط الترتيب فيه = ممكن يودي لـ Deadlock.** ده فرق مهم يتسأل فيه بصيغ مختلفة.

### Monitor:
- طريقة أعلى مستوى (high-level) لتنظيم الوصول للموارد المشتركة، بتضمن الـ Mutual Exclusion تلقائيًا (على عكس الـ Semaphore اللي المبرمج مسؤول يحطها صح).
- بيستخدم Condition Variables مع `wait()` و `signal()` لكن بمعنى مختلف عن الـ semaphore.

### مسائل الـ Semaphore الشهيرة (لازم تكون فاهمها مش حافظها):
- **Producer/Consumer** (Lab 7): Semaphore للـ dependency (مفيش عناصر تتاكل من list فاضية) + Semaphore/Mutex للـ critical section.
- **Barbershop Problem** (Lec 8): فيه Semaphores متعددة (max_capacity, sofa, barber_chair, coord, cust_ready, finished, leave_b_chair, payment, receipt). أسئلة الكويز بتسأل: "لو غيّرنا X إيه اللي هيسبب Deadlock؟" أو "لو فيه أكتر من كاشير إيه التعديل المطلوب؟" — افهم كل Semaphore بتعمل إيه بالظبط قبل ما تحاول تتوقع تأثير أي تعديل.
- **Dining Philosophers** (Lec 9 materials): مشكلة كلاسيكية للـ Deadlock + طرق تفاديها (زي: فيلسوف واحد ياخد الشوكة اليسار الأول، أو تحديد أقصى عدد فلاسفة ياكلوا في نفس الوقت).

### أسئلة متوقعة (نفس نمط الكويزات الحقيقية):
- MCQs نظرية (زي: "العملية الوحيدة المسموحة على الـ semaphore هي..." → Wait & Signal).
- كود فيه Semaphores وقيمهم الابتدائية، واسأل: "كام مرة هتطبع Process X الحرف كذا؟"
- تتبّع كود Barbershop/Dining Philosophers وتحديد: هل فيه Deadlock ولا لأ، ولو حصل تعديل معيّن هل هيسبب مشكلة.
- سؤال عن أثر ترتيب غلط لـ wait/signal أو acquire/release (زي الجدول فوق بالظبط).

---

## 6) Lecture 9 – Deadlocks

### المفاهيم الأساسية:
**4 شروط لازم تتحقق مع بعض عشان يحصل Deadlock (Coffman Conditions):**
1. **Mutual Exclusion** — المورد ميتقسمش، واحد بس ياخده.
2. **Hold and Wait** — عملية ماسكة مورد ومستنية مورد تاني.
3. **No Preemption** — المورد ميتاخدش من العملية غصب عنها.
4. **Circular Wait** — سلسلة دائرية من العمليات كل واحدة مستنية اللي بعدها.

### طرق التعامل مع الـ Deadlock:
| الطريقة | الفكرة |
|---|---|
| **Prevention** | تمنع واحد على الأقل من الـ 4 شروط من الأساس (زي: كل عملية تاخد كل مواردها قبل ما تبدأ) |
| **Avoidance** | تفحص ديناميكيًا حالة تخصيص الموارد قبل ما تدّي أي طلب — أشهرها **Banker's Algorithm** |
| **Detection** | تسيب النظام يشتغل عادي وتفحص بشكل دوري / عند كل طلب هل حصل Deadlock (باستخدام Resource Allocation Graph أو خوارزمية مشابهة لـ Banker's بس من غير منع مسبق) |
| **Recovery** | لو حصل Deadlock فعلاً: تقفل عملية أو أكتر (Abort) أو تسحب موارد (Preempt) |

### Banker's Algorithm (احفظ خطواته كويس، بييجي كـ MCQ ومسائل):
عندك لكل process: **Allocation** (اللي معاه دلوقتي) و **Max** (أقصى احتياج) و **Need = Max − Allocation**.
1. ابدأ بمتجه Available (المتاح حاليًا).
2. دوّر على process ممكن Need بتاعها ≤ Available.
3. لو لقيت، "شغّلها افتراضيًا" وضيف اللي هي ماسكاه لـ Available (Available += Allocation)، وسجّلها في الـ Safe Sequence.
4. كرر لحد ما كل العمليات تتحط في السلسلة (Safe State) أو تقف عالق (Unsafe State = ممكن يحصل Deadlock).

### Resource Allocation Graph (RAG):
- Process = دائرة، Resource = مربع.
- سهم من Process لـ Resource = طلب (Request).
- سهم من Resource لـ Process = تخصيص (Allocation).
- **لو الموارد single-instance:** دورة (cycle) في الجراف = Deadlock أكيد.
- **لو الموارد multi-instance:** دورة **مش شرط تبقى Deadlock** — لازم تتأكد فعليًا مفيش طريقة توزيع تخلص بيها كل العمليات.

### أسئلة متوقعة:
- MCQs كتير على تعريفات وشروط الـ Deadlock (زي اللي في الشيت — احفظها فهمًا مش نصًا).
- إعطاء جدول Allocation/Max/Available وسؤال: "هل السلسلة دي Safe ولا Unsafe؟" أو "رتّب Safe Sequence."
- رسمة RAG واسأل: "فيه Deadlock ولا لأ؟ ولو فيه، مين العمليات المتوقفة؟"
- مسائل تطبيقية: Dining Philosophers variations، Communication Channels.

---

## 7) Lecture 10 – Multi-CPU Scheduling

### المفاهيم الأساسية:
- **Dynamic Scheduling:** أي عملية Ready تقدر تتنفذ على أي CPU متاح (زي SPN/SRT بس على أكتر من معالج). دايمًا لو أكتر من CPU متاح، حط العملية على "أول" واحد (كما في الكويز).
- **Gang Scheduling:** مجموعة threads تابعة لنفس الـ process بيتجدولوا مع بعض على مجموعة CPUs في نفس الوقت (Uniform أو Weighted حسب الأولوية).
- **Dedicated Processor Assignment:** كل process/thread ياخد CPU مخصص طول فترة تنفيذه (زيادة أداء لكن استغلال أقل للموارد).
- **Load Sharing:** توزيع الحمل بين المعالجات من غير تخصيص صارم (FCFS مشترك بين كل الـ CPUs).

### حساب "أقصى عدد CPUs مفيد" (سؤال جالك في الكويز الحقيقي):
لو زودت عدد المعالجات لأكتر من عدد العمليات القابلة للتنفيذ بالتوازي في أي لحظة، مفيش استفادة إضافية (performance gain) — احسبها بمتابعة الـ Gantt chart وشوف أقصى عدد عمليات Ready في نفس اللحظة.

### أسئلة متوقعة:
- جدول Arrival/CPU Time لعمليات متعددة + عدد CPUs + خوارزمية (SRT/SPN)، واسأل: Finish time, حالة عملية معينة في وقت معين.
- "لو زودنا عدد المعالجات، إيه أقصى عدد مفيد قبل ما التحسّن يوقف؟"
- Gang Scheduling: توزيع Threads على مجموعات CPUs بالتساوي (Uniform) أو حسب أولوية (Weighted).

---

## 8) الـ Labs – إيه المطلوب تعرفه قبل كل معمل

### Lab 1 – البيئة والـ Kernel الأول
- تعرف على **Bochs emulator** (بتشغل عليه FOS، النظام البسيط اللي هتبنيه).
- التنقل جوه كود الـ Kernel، وإزاي تضيف أوامر بسيطة للـ Shell بتاع FOS.
- افهم عملية الـ **Boot Process** (من ملف "The PC boot process.ppt" و"Boot Loader.doc" الموجودين في مواد اللاب).
- **متوقع منك:** تعرف تضيف أمر جديد للـ Shell وتفهم شكل كود الـ Command Prompt.

### Lab 2 – Pointers و Kernel 2
- مراجعة الـ Pointers في لغة C (عنوان الذاكرة، dereferencing، pointer arithmetic) — لازم تكون متمكن منها 100% لأن باقي اللابات هتعتمد عليها.
- التعامل مع الذاكرة عبر الـ Pointers، وإضافة أوامر متقدمة للـ Kernel.

### Lab 3 – Intel Memory Management (MMU Role vs OS Role)
- إزاي معالج Intel 80386 بيترجم Virtual Address لـ Physical Address بخطوتين: **Segmentation** ثم **Paging**.
- بنية عنوان الـ Paging: **Directory | Table | Offset** (نظام 2-level على x86).
- FOS kernel space address translation و الـ Paging data structures.
- **متوقع منك:** تفهم كل خطوة في شكل الـ Figure (Selector+Offset → Segmentation → Linear Address → Paging → Physical Address)، وتربطها بمحتوى محاضرة 3 و4 نظريًا.

### Lab 4 – Managing the User Space
- 7 functions أساسية: `allocate_frame`, `free_frame`, `map_frame`, `get_page_table`, `create_page_table`, `unmap_frame`, `get_frame_info`.
- إزاي الـ Free Frame List بتشتغل، وإزاي تعمل map/unmap لعنوان افتراضي على frame معين.
- **متوقع منك:** تفهم منطق كل function ودورها في تخصيص الذاكرة، مش بس تحفظ الاسم.

### Lab 6 – Loading & Running User Programs
- الفرق بين **Program Binary** (الكود التنفيذي بعد الـ compile/link) و **Program Environment** (البنية اللي الـ Kernel بيستخدمها عشان يتابع البرنامج المحمّل: virtual address space, working set, CPU registers, ID, priority).
- إزاي FOS بيحمّل أكتر من برنامج ويشغّلهم بـ Round Robin، وإزاي يشيل برنامج من الذاكرة (إزالة الـ page table بتاعته).

### Lab 7 – Multithreading في C# (Producer/Consumer)
- الفرق بين **Thread** (وحدة تنفيذ، عندها stack خاص) و **Process** (مجموعة threads + موارد مشتركة).
- ليه محتاجين Multithreading: استغلال الـ multi-core، منع تجمّد الـ UI، تنفيذ متوازي.
- مشكلة الموارد المشتركة (Shared Resource Problem) لما أكتر من Consumer بياخدوا من نفس الـ List من غير حماية.
- حل الـ Semaphore: استخدامه للـ **Critical Section** (mutual exclusion) وللـ **Dependency** (زي منع الـ Consumer من الاستهلاك من list فاضية).
- **متوقع منك:** تعرف تكتب/تعدّل كود C# بيستخدم Semaphores لحل مشكلة Producer/Consumer، وتتوقع تأثير أي تعديل على الـ Semaphores.

### Lab 9 – Airplane Reservation (Multithreading متقدم)
- تطبيق أعمق على المشاكل الكلاسيكية (زي الموجود في مواد اللاب: Barbershop, Mergesort, Airplane Reservation).
- **متوقع منك:** تحليل نظام حجز طيران متعدد الـ Threads (كل thread بيمثل موظف حجز مثلاً) وتحديد فين ممكن يحصل Race Condition أو Deadlock، وإزاي تحلها بالـ Semaphores.

---

## 9) أسئلة "امتحانية" حقيقية جمعتها من الكويزات بتاعتك (تدرّب عليها بنفسك الأول قبل ما تشوف المفهوم تاني)

1. أي التغييرات التالية بتسبب Internal Fragmentation في جداول الصفحات؟ (فكّر في العلاقة بين حجم الـ Virtual Memory وعدد الـ entries في آخر مستوى).
2. نظام Multilevel Paging: Virtual Memory اتزودت لحجم معين — احسب أقصى بتّات للـ index وأقل عدد مستويات.
3. Trace لـ LRU مع Load-time pre-paging لصفحة معينة — احسب عدد الـ page faults وعدد الـ disk writes.
4. Gantt chart لعمليتين بأزمنة CPU/I/O مختلفة تحت FCFS — احسب Finish Time وTurnaround.
5. نظام فيه 3 Semaphores S0=1, S1=0, S2=0 وكود معين — كام مرة هتتنفذ عملية معينة؟
6. كود Barbershop — أي تعديل في ترتيب الـ semWait/semSignal ممكن يسبب Deadlock؟
7. Resource Allocation Graph مُعطى — فيه Deadlock ولا لأ؟ ولو فيه، مين العمليات المتوقفة؟
8. نظام Sleep Lock بترتيب غلط (release قبل acquire) — هيحصل إيه بالظبط؟ (قارن إجابتك بإجابة نفس السؤال لو كان Semaphore بدل الـ Lock).
9. Multi-CPU SRT scheduling على 2 معالج — احسب Finish Time لعملية معينة وحالتها في وقت معين، واحسب أقصى عدد CPUs مفيد لنفس المجموعة.
10. جدول Allocation/Max/Available — رتّب Safe Sequence أو حدد إن مفيش.

---

## 10) خلاصة استراتيجية المذاكرة للـ A+

1. **ابدأ بالمفاهيم مش بالحفظ:** كل خوارزمية (scheduling, replacement, deadlock handling) افهم *ليه* بتشتغل كده مش بس خطواتها.
2. **درّب نفسك على المسائل الحسابية بايدك** (من الـ Sheets) قبل ما تشوف الحل — دي أكتر حاجة هتتقاس بيها في الامتحان.
3. **راجع الفروق الدقيقة** اللي بتتكرر كفخاخ في الأسئلة:
   - Semaphore غلط الترتيب ≠ Lock غلط الترتيب.
   - زيادة الـ Virtual Memory تأثيرها عكس زيادة الـ Physical Memory على الـ Internal Fragmentation.
   - Deadlock بشروطه الأربعة مع بعض، مش شرط واحد بس.
   - Cycle في RAG = Deadlock أكيد بس لو الموارد single-instance، مش لو multi-instance.
4. **اربط كل Lecture بالـ Lab المقابل لها** — لأن اللاب بيوضحلك تطبيق عملي لنفس المفهوم النظري (زي Lab3/4/6 مع Lectures 3/4/5، وLab7/9 مع Lectures 7/8).
5. **راجع شكل الامتحان الحقيقي** (الميدل والـ Retake في ملفاتك) قبل الامتحان بيوم، عشان تتعود على شكل الأسئلة بالظبط.

بالتوفيق! 💪
