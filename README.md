# kaggle_titanic
Source code for Kaggle Titanic Competition

# 1. 데이터 분석
## 1.1 변수 종류 확인
- Survied 포함 총 12개의 Variable (제외하고는 11개의 variable)
- Survived 변수는 정답 label로써
  - train set에는 포함
  - test set에는 미포함
```
> train_df.columns
```
```
    Index(['PassengerId', 'Survived', 'Pclass', 'Name', 'Sex', 'Age', 'SibSp',
        'Parch', 'Ticket', 'Fare', 'Cabin', 'Embarked'],
        dtype='object')
```

## 1.2 Column별 데이터 분석
- 각 Column들의 데이터 종류를 확인한다
```
for col in train_df:
    print(train_df[col].value_counts())
    print()
```
```
891    1
293    1
304    1
303    1
302    1
      ..
591    1
590    1
589    1
588    1
1      1
Name: PassengerId, Length: 891, dtype: int64

0    549
1    342
Name: Survived, dtype: int64

3    491
1    216
2    184
Name: Pclass, dtype: int64

McGowan, Miss. Anna "Annie"                1
Hakkarainen, Mr. Pekka Pietari             1
Nakid, Mr. Sahid                           1
Myhrman, Mr. Pehr Fabian Oliver Malkolm    1
Blackwell, Mr. Stephen Weart               1
                                          ..
Cacic, Mr. Luka                            1
Barbara, Miss. Saiide                      1
Ilett, Miss. Bertha                        1
West, Mr. Edwy Arthur                      1
Chambers, Mr. Norman Campbell              1
Name: Name, Length: 891, dtype: int64

male      577
female    314
Name: Sex, dtype: int64

24.00    30
22.00    27
18.00    26
19.00    25
30.00    25
         ..
55.50     1
70.50     1
66.00     1
23.50     1
0.42      1
Name: Age, Length: 88, dtype: int64

0    608
1    209
2     28
4     18
3     16
8      7
5      5
Name: SibSp, dtype: int64

0    678
1    118
2     80
5      5
3      5
4      4
6      1
Name: Parch, dtype: int64

1601          7
347082        7
CA. 2343      7
3101295       6
CA 2144       6
             ..
347060        1
36967         1
349912        1
350035        1
C.A. 31026    1
Name: Ticket, Length: 681, dtype: int64

8.0500     43
13.0000    42
7.8958     38
7.7500     34
26.0000    31
           ..
8.4583      1
9.8375      1
8.3625      1
14.1083     1
17.4000     1
Name: Fare, Length: 248, dtype: int64

G6             4
C23 C25 C27    4
B96 B98        4
F33            3
E101           3
              ..
D49            1
B69            1
A31            1
A32            1
C30            1
Name: Cabin, Length: 147, dtype: int64

S    644
C    168
Q     77
Name: Embarked, dtype: int64
```
- 각 Column Data에 Nan값이 들어 있는지를 확인한다
```
# NaN값이 들어있는 columns를 확인한다
train_df.isnull().sum(axis=0)
```
```
PassengerId      0
Survived         0
Pclass           0
Name             0
Sex              0
Age            177
SibSp            0
Parch            0
Ticket           0
Fare             0
Cabin          687
Embarked         2
dtype: int64
```
- Age, Cabin, Embarked에 NaN값들이 있는 것을 볼 수 있다.
  - 이 값을 어떻게 Preprocessing 하는지가 문제!
      - 단순히 Age의 평균으로 채우는 방법
      - 다른 변수들과의 관계를 고려해 채우는 방법
        - ex)
        - Parch가 높을수록 Age는 높더라 이런식? (예시일 뿐!)
  - 우리는 단순히 Age의 평균을 이용해서 NaN을 채우는 방법을 이용할 것

</br>

## 1.2 관련성 없는 변수들 삭제
- 그냥 쳐다봤을 때 관련이 없어 보이는 변수들이 존재
- 이 변수들을 삭제하고 NumPy 배열로 변환
- 총 다섯 가지:
  - Ticket
  - Fare
  - Cabin
  - PassengerId
  - Name
- Feature Engineering을 이용해 이 특성들을 이용할 방법도 있지만, 현 단계에서는 일단 배제하기로 함
  
```
    train = train_df.drop(["Ticket", "Fare", "Cabin", "PassengerId", "Name"], axis=1, )
    test = test_df.drop(["Ticket", "Fare", "Cabin", "PassengerId", "Name"], axis=1, )
```

## 1.3 Data PreProcessing
0. NaN 데이터 처리하기
- pandas.DataFrame.fillna() 를 이용해 Age의 NaN 값을 채울 수 있다
```
train["Age"].fillna(train["Age"].mean(), inplace=True)
test["Age"].fillna(train["Age"].mean(), inplace=True)
```
- 중요한 점은, 
- **Test Dataset에 Train Dataset을 이용한 처리**를 해주어야 한다는 점이다
  - 그렇지 않으면 데이터의 통일성이 떨이지게 됨
  - 항상 데이터에 대한 처리를 수행할 때에는 각 DataSet에 동일하게 적용할 것
  - 말이 이상하기는 한데 암튼...

1. Sex : One-hot Encoding
```
# male, female 값을 각각 0과 1로 인코딩한다
sex_i = 3
embark_i = 7
for data in set:
    sex_i -= 1
    embark_i -= 1
    for i in range(data.shape[0]):
        if data[i,sex_i] == 'male' :
            data[i,sex_i] = 0
        else :
            data[i,sex_i] = 1
        if data[i,embark_i] == 'S':
            data[i,embark_i] = 0
        elif data[i,embark_i] == 'Q':
            data[i,embark_i] = 1
        else :
            data[i,embark_i] = 2
```
2. Data Type Change : int32? --> float64
    - 사유 : 텐서플로를 벡엔드로 사용하는 Keras는 int를 입력으로 받지 않음
```
# 데이터 형변환
train = train.astype(np.float64)
test = test.astype(np.float64)
```
3. Training Data 준비
```
# 훈련용 변수와 정답 레이블을 분리한다
x_train = train[:,1:]
y_train = train[:,0]
```

# 2. Keras 모델 설계
```
# keras모델 설계
model = models.Sequential()
model.add(layers.Dense(16, activation="relu", input_shape=(6,)))
model.add(layers.Dropout(0.3))
model.add(layers.Dense(16, activation="relu"))
model.add(layers.Dropout(0.3))
model.add(layers.Dense(1, activation="sigmoid"))

model.compile(optimizer="rmsprop", loss="binary_crossentropy", metrics=["binary_accuracy"])
```
```
history = model.fit(
    x_train, y_train, epochs=200, batch_size=20, validation_split=0.2, shuffle = True, verbose=1
)

history_dict = history.history
history_dict.keys()
```
# 3. 결과 분석
```
acc = history.history['binary_accuracy']
val_acc = history.history['val_binary_accuracy']
loss = history.history['loss']
val_loss = history.history['val_loss']

epochs = range(1, len(acc) + 1)

# ‘bo’는 파란색 점을 의미합니다
plt.plot(epochs, loss, 'bo', label='Training loss')
# ‘b’는 파란색 실선을 의미합니다
plt.plot(epochs, val_loss, 'b', label='Validation loss')
plt.title('Training and validation loss')
plt.xlabel('Epochs')
plt.ylabel('Loss')
plt.legend()

plt.show()
```
- 결과 :
- ![accuracy](img/accuracy.png)
```
plt.clf()   # 그래프를 초기화합니다
acc = history_dict['binary_accuracy']
val_acc = history_dict['val_binary_accuracy']

plt.plot(epochs, acc, 'bo', label='Training acc')
plt.plot(epochs, val_acc, 'b', label='Validation acc')
plt.title('Training and validation accuracy')
plt.xlabel('Epochs')
plt.ylabel('Accuracy')
plt.legend()

plt.show()
```
- 결과 :
- ![loss](img/loss.png)

# 4. Train 데이터 예측 / submission.csv 생성
- predict() 수행 후,
- 결과 (result)를 1차원으로 변환
```
result = model.predict(test)
result = result.reshape(418)
# result.shape
```
- submission.csv의 형식과 동일하도록 pandas를 이용해 DataFrame을 생성한 후 저장
```
my_submission = pd.DataFrame({'PassengerId': test_df.PassengerId, 'Survived': result})
my_submission.to_csv('submission.csv', index=False)
```
# 5. 배운 점
- Data PreProcessing은 매우 중요하다
  - Kaggle에는 보통 잘 정제된 Data들이 올라오는 편
  - 이러한 데이터들에도 NaN과 같은 데이터들이 있는데,
  - 변수들간의 상간관계를 분석해서 이러한 결측치들을 어떻게 채울지 등이 중요!
- Pandas, NumPy 라이브러리에 좀 더 익숙해질 필요성을 느낀다
- 다른 사람들 NoteBook 보고 더 많이 공부해야겠다 ㅜㅜㅜ
