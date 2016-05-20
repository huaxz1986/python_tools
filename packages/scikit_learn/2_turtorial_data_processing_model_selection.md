# 模型参数的选择和评价标准

## 一、score

1.每一个 `estimator` 都有一个 `.score(x,y)`方法，它给出了模型的预测质量。该值越大，则模型的能力越强。其中 `(x,y)`为训练集或者测试集

```
from sklearn import datasets, svm
digits = datasets.load_digits()
X_digits = digits.data
y_digits = digits.target
svc = svm.SVC(C=1, kernel='linear')
svc.fit(X_digits[:-100], y_digits[:-100]).score(X_digits[-100:], y_digits[-100:])
```
2.通常为了更准确的测试模型的能力，我们采取 K 折交算法

```
# 3 折交算法
import numpy as np
X_folds = np.array_split(X_digits, 3)
y_folds = np.array_split(y_digits, 3)
scores = list()
for k in range(3):
     # We use 'list' to copy, in order to 'pop' later on
     X_train = list(X_folds)
     X_test  = X_train.pop(k)
     X_train = np.concatenate(X_train)
     y_train = list(y_folds)
     y_test  = y_train.pop(k)
     y_train = np.concatenate(y_train)
     scores.append(svc.fit(X_train, y_train).score(X_test, y_test))
print(scores)
```

当然我们也可以使用 `scikit-learn` 提供的函数：

```
from sklearn import cross_validation
k_fold = cross_validation.KFold(n=6, n_folds=3)
for train_indices, test_indices in k_fold:
      print('Train: %s | test: %s' % (train_indices, test_indices))
kfold = cross_validation.KFold(len(X_digits), n_folds=3)
 [svc.fit(X_digits[train], y_digits[train]).score(X_digits[test], y_digits[test])\
 for train, test in kfold]

```

`sklearn`提供了一个便利函数用来简化这一流程：

```
from sklearn import datasets, svm
digits = datasets.load_digits()
X_digits = digits.data
y_digits = digits.target
svc = svm.SVC(C=1, kernel='linear')
from sklearn import cross_validation
kfold = cross_validation.KFold(len(X_digits), n_folds=3)
cross_validation.cross_val_score(svc, X_digits, y_digits, cv=kfold, n_jobs=-1)
```

其中 `n_jobs=-1` 意思是计算会派发到所有计算机的 CPU 中进行。

3.常用的折交集合生成器：

- `KFold(n,k)`:Split it K folds, train on K-1 and then test on left-out
- `StratifiedKFold (y, k)`：It preserves the class ratios / label distribution within each fold.
- `LeaveOneOut (n)`：Leave one observation out
- `LeaveOneLabelOut (labels)`：Takes a label array to group observations

```
import numpy as np
from sklearn import cross_validation, datasets, svm

digits = datasets.load_digits()
X = digits.data
y = digits.target

svc = svm.SVC(kernel='linear')
C_s = np.logspace(-10, 0, 10)

scores = list()
scores_std = list()
for C in C_s:
    svc.C = C
    this_scores = cross_validation.cross_val_score(svc, X, y, n_jobs=1)
    scores.append(np.mean(this_scores))
    scores_std.append(np.std(this_scores))

# Do the plotting
import matplotlib.pyplot as plt
plt.figure(1, figsize=(4, 3))
plt.clf()
plt.semilogx(C_s, scores)
plt.semilogx(C_s, np.array(scores) + np.array(scores_std), 'b--')
plt.semilogx(C_s, np.array(scores) - np.array(scores_std), 'b--')
locs, labels = plt.yticks()
plt.yticks(locs, list(map(lambda x: "%g" % x, locs)))
plt.ylabel('CV score')
plt.xlabel('Parameter C')
plt.ylim(0, 1.1)
plt.show()
```
## 二、grid search

1.`sklearn` 提供了一个对象，用于搜索最佳参数。当提供数据集时，该对象能够自动训练模型，然后选择最大化的`cross-validation score`时的参数。

```
from sklearn.grid_search import GridSearchCV
Cs = np.logspace(-6, -1, 10)
clf = GridSearchCV(estimator=svc, param_grid=dict(C=Cs), n_jobs=-1)
clf.fit(X_digits[:1000], y_digits[:1000])        
print(clf.best_score_  )                                
print(clf.best_estimator_.C )                           
# Prediction performance on test set is not as good as on train set
clf.score(X_digits[1000:], y_digits[1000:])    
```

默认情况下 `GridSearchCV` 使用 3折交叉验证。但是如果它检测到传入的是一个 `classifier`而不是 `regressor`，则它会使用 `stratified 3-fold`

2.Nested cross-validation

```
cross_validation.cross_val_score(clf, X_digits, y_digits)
```

Two cross-validation loops are performed in parallel: one by the GridSearchCV estimator to set gamma and the other one by cross_val_score to measure the prediction performance of the estimator. The resulting scores are unbiased estimates of the prediction score on new data.


3.Cross-validation to set a parameter can be done more efficiently on an algorithm-by-algorithm basis. This is why for certain estimators the sklearn exposes Cross-validation: evaluating estimator performance estimators that set their parameter automatically by cross-validation:

```
from sklearn import linear_model, datasets
lasso = linear_model.LassoCV()
diabetes = datasets.load_diabetes()
X_diabetes = diabetes.data
y_diabetes = diabetes.target
lasso.fit(X_diabetes, y_diabetes)
print(lasso.alpha_) 
```
These estimators are called similarly to their counterparts, with ‘CV’ appended to their name.