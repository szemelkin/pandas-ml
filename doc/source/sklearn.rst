
Use scikit-learn
================

This section describes how to use ``scikit-learn`` functionalities via ``expandas``.

Basics
------

You can create ``ModelFrame`` instance from ``scikit-learn`` datasets directly.

.. code-block:: python

   >>> import expandas as expd
   >>> import sklearn.datasets as datasets

   >>> df = expd.ModelFrame(datasets.load_iris())
   >>> df.head()
      .target  sepal length (cm)  sepal width (cm)  petal length (cm)  \
   0        0                5.1               3.5                1.4
   1        0                4.9               3.0                1.4
   2        0                4.7               3.2                1.3
   3        0                4.6               3.1                1.5
   4        0                5.0               3.6                1.4

      petal width (cm)
   0               0.2
   1               0.2
   2               0.2
   3               0.2
   4               0.2

   # make columns be readable
   >>> df.columns = ['.target', 'sepal length', 'sepal width', 'petal length', 'petal width']

``ModelFrame`` has accessor methods which makes easier access to ``scikit-learn`` namespace.

.. code-block:: python

   >>> df.cluster.KMeans
   <class 'sklearn.cluster.k_means_.KMeans'>

Following table shows ``scikit-learn`` module and corresponding ``ModelFrame`` module.

================================  ================================
``scikit-learn``                  ``ModelFrame`` accessor
================================  ================================
``sklearn.cluster``               ``ModelFrame.cluster``
``sklearn.covariance``            ``ModelFrame.covariance``
``sklearn.cross_validation``      ``ModelFrame.cross_validation``
``sklearn.decomposition``         ``ModelFrame.decomposition``
``sklearn.datasets``              (not accesible from accessor)
``sklearn.dummy``                 ``ModelFrame.dummy``
``sklearn.ensemble``              ``ModelFrame.ensemble``
``sklearn.feature_extraction``
``sklearn.gaussian_process``
``sklearn.grid_search``           ``ModelFrame..grid_search``
``sklearn.isotonic``
``sklearn.kernel_approximation``
``sklearn.lda``                   ``ModelFrame.lda``
``sklearn.linear_model``          ``ModelFrame.linear_model``
``sklearn.manifold``
``sklearn.metrics``               ``ModelFrame..metrics``
``sklearn.mixture``
``sklearn.multiclass``
``sklearn.naive_bayes``           ``ModelFrame..naive_bayes``
``sklearn.neighbors``             ``ModelFrame..neighbors``
``sklearn.cross_decomposition``
``sklearn.pipeline``              ``ModelFrame.pipeline``
``sklearn.preprocessing``         ``ModelFrame.preprocessing``
``sklearn.qda``
``sklearn.semi_supervised``
``sklearn.svm``                   ``ModelFrame.svm``
``sklearn.tree``
``sklearn.utils``
================================  ================================

Thus, you can instanciate each estimator via ``ModelFrame`` accessors. Once create an estimator, you can pass it to ``ModelFrame.fit`` then ``predict``. `ModelFrame`` automatically uses its data and target properties for each operations.

.. code-block:: python

   >>> estimator = df.cluster.KMeans(n_clusters=3)
   >>> df.fit(estimator)

   >>> predicted = df.predict(estimator)
   >>> predicted
   0    1
   1    1
   2    1
   ...
   147    2
   148    2
   149    0
   Length: 150, dtype: int32

``ModelFrame`` has following methods corresponding to various ``scikit-learn`` estimators.

- ``ModelFrame.fit``
- ``ModelFrame.transform``
- ``ModelFrame.fit_transform``
- ``ModelFrame.inverse_transform``
- ``ModelFrame.predict``
- ``ModelFrame.fit_predict``
- ``ModelFrame.score``

Following example shows to perform PCA.

.. code-block:: python

   >>> estimator = df.decomposition.PCA()
   >>> df.fit(estimator)

   >>> transformed = df.transform(estimator)
   >>> transformed.head()
      .target         0         1         2         3
   0        0 -2.684207 -0.326607  0.021512  0.001006
   1        0 -2.715391  0.169557  0.203521  0.099602
   2        0 -2.889820  0.137346 -0.024709  0.019305
   3        0 -2.746437  0.311124 -0.037672 -0.075955
   4        0 -2.728593 -0.333925 -0.096230 -0.063129

   >>> type(transformed)
   <class 'expandas.core.frame.ModelFrame'>

   >>> transformed.inverse_transform(estimator)
        .target    0    1    2    3
   0          0  5.1  3.5  1.4  0.2
   1          0  4.9  3.0  1.4  0.2
   2          0  4.7  3.2  1.3  0.2
   3          0  4.6  3.1  1.5  0.2
   4          0  5.0  3.6  1.4  0.2
   ..       ...  ...  ...  ...  ...
   145        2  6.7  3.0  5.2  2.3
   146        2  6.3  2.5  5.0  1.9
   147        2  6.5  3.0  5.2  2.0
   148        2  6.2  3.4  5.4  2.3
   149        2  5.9  3.0  5.1  1.8

   [150 rows x 5 columns]


.. note:: ``columns`` information will be lost once transformed to principal components.

``ModelFrame`` preserves last predicted result in ``predicted`` attibute. If ``ModelFrame`` both has ``target`` and ``predicted`` values, the model evaluation can be performed using functions available in ``ModelFrame.metrics``.

.. code-block:: python

   >>> estimator = df.svm.SVC()
   >>> df.fit(estimator)

   >>> df.predict(estimator)
   0    0
   1    0
   2    0
   ...
   147    2
   148    2
   149    2
   Length: 150, dtype: int64

   >>> df.predicted
   0    0
   1    0
   2    0
   ...
   147    2
   148    2
   149    2
   Length: 150, dtype: int64

   >>> df.metrics.confusion_matrix()
       0   1   2
   0  50   0   0
   1   0  48   2
   2   0   0  50

Pipeline
--------

``ModelFrame`` can handle pipeline as the same as normal estimators.

.. code-block:: python

   >>> estimators = [('reduce_dim', df.decomposition.PCA()),
   ...               ('svm', df.svm.SVC())]
   >>> pipe = df.pipeline.Pipeline(estimators)
   >>> df.fit(pipe)

   >>> df.predict(pipe)
   0    0
   1    0
   2    0
   ...
   147    2
   148    2
   149    2
   Length: 150, dtype: int64

Above expression is the same as below:

.. code-block:: python

   >>> df2 = df.copy()
   >>> df2 = df2.fit_transform(df2.decomposition.PCA())
   >>> svm = df2.svm.SVC()
   >>> df2.fit(svm)
   SVC(C=1.0, cache_size=200, class_weight=None, coef0=0.0, degree=3, gamma=0.0,
     kernel='rbf', max_iter=-1, probability=False, random_state=None,
     shrinking=True, tol=0.001, verbose=False)
   >>> df2.predict(svm)
   0     0
   1     0
   2     0
   ...
   147    2
   148    2
   149    2
   Length: 150, dtype: int64


Cross Validation
----------------

``scikit-learn`` has some classes for cross validation. ``cross_validation.train_test_split`` splits data to training and test set. You can access to the function via ``cross_validation`` accessor.

.. code-block:: python

   >>> train_df, test_df = df.cross_validation.train_test_split()
   >>> train_df
        .target  sepal length  sepal width  petal length  petal width
   0          0           4.8          3.4           1.9          0.2
   1          1           6.3          3.3           4.7          1.6
   2          0           4.8          3.4           1.6          0.2
   3          2           7.7          2.6           6.9          2.3
   4          0           5.4          3.4           1.7          0.2
   ..       ...           ...          ...           ...          ...
   107        0           5.1          3.7           1.5          0.4
   108        1           6.7          3.1           4.7          1.5
   109        0           4.7          3.2           1.3          0.2
   110        0           5.8          4.0           1.2          0.2
   111        0           5.1          3.5           1.4          0.2

   [112 rows x 5 columns]

   >>> test_df
       .target  sepal length  sepal width  petal length  petal width
   0         2           6.3          2.7           4.9          1.8
   1         0           4.5          2.3           1.3          0.3
   2         2           5.8          2.8           5.1          2.4
   3         0           4.3          3.0           1.1          0.1
   4         0           5.0          3.0           1.6          0.2
   ..      ...           ...          ...           ...          ...
   33        1           6.7          3.1           4.4          1.4
   34        0           4.6          3.6           1.0          0.2
   35        1           5.7          3.0           4.2          1.2
   36        1           5.9          3.0           4.2          1.5
   37        2           6.4          2.8           5.6          2.1

   [38 rows x 5 columns]


Also, there are some iterative classes which returns indexes for training sets and test sets. You can slice ``ModelFrame`` using these indexes.

.. code-block:: python

   >>> kf = df.cross_validation.KFold(n=150, n_folds=3)
   >>> for train_index, test_index in kf:
   ...    print('training set shape: ', df.iloc[train_index, :].shape,
   ...          'test set shape: ', df.iloc[test_index, :].shape)
   ('training set shape: ', (100, 5), 'test set shape: ', (50, 5))
   ('training set shape: ', (100, 5), 'test set shape: ', (50, 5))
   ('training set shape: ', (100, 5), 'test set shape: ', (50, 5))


For further simplification, ``ModelFrame.cross_validation.iterate`` can accept such iterators and returns ``ModelFrame`` corresponding to training and test data.

.. code-block:: python

   >>> kf = df.cross_validation.KFold(n=150, n_folds=3)
   >>> for train_df, test_df in df.cross_validation.iterate(kf):
   ...    print('training set shape: ', train_df.shape,
   ...          'test set shape: ', test_df.shape)
   ('training set shape: ', (100, 5), 'test set shape: ', (50, 5))
   ('training set shape: ', (100, 5), 'test set shape: ', (50, 5))
   ('training set shape: ', (100, 5), 'test set shape: ', (50, 5))

Grid Search
-----------

You can perform grid search using ``ModelFrame.fit``.

.. code-block:: python

   >>> tuned_parameters = [{'kernel': ['rbf'], 'gamma': [1e-3, 1e-4],
   ...                     'C': [1, 10, 100]},
   ...                    {'kernel': ['linear'], 'C': [1, 10, 100]}]

   >>> df = expd.ModelFrame(datasets.load_digits())
   >>> cv = df.grid_search.GridSearchCV(df.svm.SVC(C=1), tuned_parameters,
   ...                                  cv=5, scoring='precision')

   >>> df.fit(cv)

   >>> cv.best_estimator_
   SVC(C=10, cache_size=200, class_weight=None, coef0=0.0, degree=3, gamma=0.001,
     kernel='rbf', max_iter=-1, probability=False, random_state=None,
     shrinking=True, tol=0.001, verbose=False)

In addition, ``ModelFrame.grid_search`` has a ``describe`` function to organize each grid search result as ``pd.DataFrame`` accepting estimator.

.. code-block:: python

   >>> df.grid_search.describe(cv)
          mean       std    C   gamma  kernel
   0  0.974108  0.013139    1  0.0010     rbf
   1  0.951416  0.020010    1  0.0001     rbf
   2  0.975372  0.011280   10  0.0010     rbf
   3  0.962534  0.020218   10  0.0001     rbf
   4  0.975372  0.011280  100  0.0010     rbf
   5  0.964695  0.016686  100  0.0001     rbf
   6  0.951811  0.018410    1     NaN  linear
   7  0.951811  0.018410   10     NaN  linear
   8  0.951811  0.018410  100     NaN  linear