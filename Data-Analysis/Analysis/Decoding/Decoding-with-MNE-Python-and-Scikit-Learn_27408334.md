# MEG : Decoding with MNE Python and Scikit-Learn

Scikit-learn is a free and powerful software machine learning library in Python. It can be used with MNE Python for decoding and classification of EEG/MEG data.

Below is a typical decoding pipeline.

Note: this page is still under construction. Installing Python and MNE dependencies, as well as importing data into Python will be discussed in another section.

------------------------------------------------------------------------------

In this experiment, I want to decode left vs right orientation of a stimulus, by training a classifier on a specific dataset and testing it on another

------------------------------------------------------------------------------

#start with MNE epochs labelled 'Train' and 'Test'

X_train = epochs['Train'].get_data()  # MEG signals: n_epochs, n_channels, n_times

y_train = epochs['Train'].events[:, 2]  # codes left or right

X_test =  epochs['Test'].get_data()  # MEG signals: n_epochs, n_channels, n_times

y_test = epochs['Test'].events[:, 2] # codes left or right

clf = make_pipeline(StandardScaler(), LinearModel(LogisticRegression()))   #use a logistic regression classifier

time_decod = SlidingEstimator(clf, n_jobs=1, scoring='roc_auc')   # use a sliding (across time) estimator and AUC measure for scoring

scores = time_decod.fit(X_train, y_train).score(X_test, y_test)  # fit model on train data and score on test data
