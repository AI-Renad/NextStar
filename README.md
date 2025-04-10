# NextStar
مرحبًا جميعًا،
أود أن أقدم لكم ملف البيانات الذي يحتوي على معلومات شاملة حول أداء اللاعبين في كرة القدم. يتضمن هذا الملف تقييمات اللاعبين في مختلف المراكز ومجموعة من الميزات الأساسية التي تساعدنا في تحليل الأداء بدقة.
محتويات الملف:
أسماء اللاعبين: قائمة بجميع اللاعبين المدروسة.
النوادي: الأندية التي ينتمي إليها اللاعبون.
تقييمات الأداء: تشمل تقييمات عامة وتخصصية مثل السرعة، التمرير، المراوغة، والدفاع.
بيانات عن المراكز: معلومات حول المراكز المختلفة التي يلعبون فيها.
هذا الملف سيكون أساس التحليل الذي نقدمته حول أداء اللاعبين واحتياجات الأندية. يمكنك الوصول إلى الملف عبر هذا الرابط: https://www.kaggle.com/code/aryantiwari123/fifa-19-complete-player-dataset.


في عالم كرة القدم المعاصر، تعتبر تحليلات الأداء أداة حيوية لتحقيق النجاح المداري وتعزيز استراتيجيات الفرق. يتناول هذا التقرير تحليل أداء اللاعبين والأندية، مستندًا إلى مجموعة من البيانات الشاملة التي تضم تقييمات اللاعبين واحتياجات الأندية في المراكز المختلفة. من خلال استخدام أدوات وتقنيات تحليل البيانات، نستهدف فهم العلاقة بين أداء اللاعبين واحتياجات فرقهم.


# تحليل أداء اللاعبين والأندية

## 1. أكثر اللاعبين تشابهًا مع ميسي

```python
import pandas as pd

# تحميل البيانات
file_path = '/Users/renad/Desktop/kl.csv'  
df = pd.read_csv(file_path, encoding='ISO-8859-1')  # لتجنب خطأ Unicode

# الأعمدة التي نعتمد عليها لتقييم اللاعبين
features = [
    'Overall', 'Potential',
    'Acceleration', 'SprintSpeed',  # سرعة
    'Finishing', 'ShotPower', 'LongShots',  # تسديد
    'ShortPassing', 'LongPassing', 'Vision',  # تمرير
    'Dribbling', 'BallControl',  # مراوغة
    'StandingTackle', 'SlidingTackle', 'Marking',  # دفاع
    'Strength', 'Stamina', 'Aggression', 'Jumping'  # بدني
]

# حذف الصفوف التي تحتوي على بيانات ناقصة
df = df.dropna(subset=features)

# فلترة لاعب عالمي (مثل ميسي)
messi = df[df['Name'].str.contains("Messi", case=False)].iloc[0]

# حساب الفرق بين اللاعبين وميسي
df['Similarity_to_Messi'] = df[features].apply(lambda row: ((row - messi[features])**2).sum(), axis=1)

# ترتيب اللاعبين حسب التشابه مع ميسي
similar_players = df.sort_values(by='Similarity_to_Messi').head(10)

print("أكثر اللاعبين تشابهًا مع ميسي:")
print(similar_players[['Name', 'Club', 'Overall', 'Potential']])


```

![image](https://github.com/user-attachments/assets/c2c38340-7e3e-4bac-b67d-dfd647bac24c)


# 2. تحليل احتياجات الأندية:
```python
import pandas as pd

# مراكز اللاعبين الرئيسية
positions = ['ST', 'CB', 'CM', 'CAM', 'CDM', 'LB', 'RB', 'GK']

# حذف الصفوف التي بدون مركز أو نادي
df = df.dropna(subset=['Position', 'Club', 'Overall'])

# حصر فقط اللاعبين في المراكز المعروفة
df = df[df['Position'].isin(positions)]

# نحسب متوسط الأداء لكل نادي في كل مركز
club_needs = df.groupby(['Club', 'Position'])['Overall'].mean().reset_index()

# Pivot Table لتحليل سهل
pivot_table = club_needs.pivot(index='Club', columns='Position', values='Overall')

# حساب المتوسط العام لكل نادي
pivot_table['Overall Avg'] = pivot_table.mean(axis=1)

# حساب الفرق بين كل مركز وبين المتوسط العام للنادي
for pos in positions:
    pivot_table[f'{pos}_Gap'] = pivot_table['Overall Avg'] - pivot_table[pos]

# عرض الأندية التي لديها ضعف واضح في مركز معين
gaps = pivot_table[[f'{pos}_Gap' for pos in positions]]
weakness_summary = gaps[gaps > 5].dropna(how='all')

print("الأندية التي لديها ضعف في مراكز معينة (أكبر من 5 نقاط فرق):")
print(weakness_summary)
```
![image](https://github.com/user-attachments/assets/996455eb-bcd2-4168-bf01-c400dc088bc2)

### 3. اقتراحات للاعبين لتحسين الأداء

```python
# اقتراح اللاعبين المناسبين للمراكز الضعيفة:

# اختيار النادي الذي لديه ضعف في مركز معين
target_club = 'FC Barcelona'
weak_position = 'CB'

# تحديد مستوى الأداء المتوسط للنادي في هذا المركز
club_avg = pivot_table.loc[target_club, weak_position]

# اختيار لاعبين بنفس المركز ومستواهم أعلى من المتوسط
recommended_players = df[
    (df['Position'] == weak_position) &
    (df['Overall'] > club_avg) &
    (df['Club'] != target_club)  # استثناء لاعبين نفس النادي
]

# ترتيبهم حسب الأداء
recommended_players = recommended_players.sort_values(by='Overall', ascending=False)

# عرض أول 10 لاعبين مرشحين
print(f"\n🔍 اقتراحات للاعبين في مركز {weak_position} لتحسين أداء نادي {target_club}:\n")
print(recommended_players[['Name', 'Age', 'Nationality', 'Overall', 'Potential', 'Club']].head(10))

![image](https://github.com/user-attachments/assets/77083478-0645-4e0f-821a-b92580f3974f)
```

# 4. اقتراح لاعبين لأكثر من مركز ضعيف في نادي معين
```python
# تحديد اسم النادي
target_club = 'FC Barcelona'

# تحديد عدد المراكز التي نعتبرها "ضعف" (مثلاً أضعف 3 مراكز في النادي)
num_weak_positions = 3

# الحصول على تقييمات اللاعبين في كل مركز داخل النادي
club_player_data = df[df['Club'] == target_club]
club_position_avg = club_player_data.groupby('Position')['Overall'].mean().sort_values()

# اختيار أضعف المراكز
weak_positions = club_position_avg.head(num_weak_positions).index.tolist()

# طباعة المراكز الضعيفة
print(f"\n📉 أضعف المراكز في نادي {target_club}: {weak_positions}")

# اقتراح لاعبين لكل مركز
for position in weak_positions:
    avg = club_position_avg[position]
    recommended_players = df[
        (df['Position'] == position) &
        (df['Overall'] > avg) &
        (df['Club'] != target_club)
    ].sort_values(by='Overall', ascending=False)

    print(f"\n اقتراحات للاعبين في مركز {position} لتحسين أداء نادي {target_club}:\n")
    print(recommended_players[['Name', 'Age', 'Nationality', 'Overall', 'Potential', 'Club']].head(5))
```
![image](https://github.com/user-attachments/assets/c04ed31e-5e73-4122-a9fb-afb3a753da2c)
```python
# تحديد المهارات المهمة للاعبين الوسط
features = ['ShortPassing', 'Vision', 'BallControl', 'Overall', 'Potential', 'Position', 'Name', 'Age', 'Club', 'Nationality']

# حذف الصفوف التي تحتوي على بيانات ناقصة في الأعمدة المطلوبة
df_clean = df[features].dropna()

# فلترة لاعبين الوسط فقط
midfielders = df_clean[df_clean['Position'].isin(['CM', 'CAM', 'CDM'])]

# حساب متوسط لكل لاعب بناءً على المهارات الأساسية
midfielders['TalentScore'] = midfielders[['ShortPassing', 'Vision', 'BallControl', 'Overall', 'Potential']].mean(axis=1)

# عرض أفضل 10 لاعبين للتعاقد
top_recommendations = midfielders.sort_values(by='TalentScore', ascending=False).head(10)

print("أفضل التوصيات للتعاقد مع لاعبي وسط:")
print(top_recommendations[['Name', 'Club', 'Age', 'Nationality', 'TalentScore']])
```
![image](https://github.com/user-attachments/assets/6438c532-e12b-4c46-b9cd-21df396d2060)

### 7. تحليل شامل للأداء

```python
# تحليل شامل للأداء عبر مختلف الإحصائيات
import matplotlib.pyplot as plt
import seaborn as sns

# اختيار بعض الإحصائيات الرئيسية
selected_features = ['Overall', 'Potential', 'Acceleration', 'SprintSpeed', 'Finishing']

# رسم بياني لتوزيع تقييمات اللاعبين في النادي
plt.figure(figsize=(12, 6))
sns.boxplot(data=df[selected_features])
plt.title('توزيع تقييمات اللاعبين في النادي')
plt.xlabel('الإحصائيات')
plt.ylabel('التقييمات')
plt.show()
```
![image](https://github.com/user-attachments/assets/45658e2e-95f1-4d0c-adda-871ab1d8225e)

# تحليل العلاقة بين القدرات المختلفة
```python
plt.figure(figsize=(10, 6))
sns.heatmap(df[selected_features].corr(), annot=True, fmt=".2f", cmap='coolwarm')
plt.title('المصفوفة الحرارية للعلاقات بين القدرات المختلفة')
plt.show()
```
![image](https://github.com/user-attachments/assets/dfafe2dc-6caa-4160-b594-ff41311736c7)


------------------------------------------------------------------
الخاتمة

في الختام، توفر تحليل أداء اللاعبين واحتياجات الأندية رؤى قيمة يمكن أن تساعد بشكل كبير في اتخاذ القرارات الاستراتيجية ضمن عالم كرة القدم. من خلال الاستفادة الفعالة من البيانات، يمكن للفرق تحسين عمليات التوظيف، وتعزيز أداء اللاعبين، ومعالجة نقاط الضعف المحددة في تشكيلاتها. تُعد التقييمات المستمرة والتكيف بناءً على مقاييس الأداء أمرًا حيويًا للحفاظ على ميزة تنافسية في هذه الرياضة الديناميكية.
شكرًا لاهتمامكم، وأتطلع إلى أي أسئلة أو مناقشات قد تكون لديكم!
