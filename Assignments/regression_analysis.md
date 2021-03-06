{
 "cells": [
  {
   "cell_type": "code",
   "execution_count": 1,
   "metadata": {},
   "outputs": [
    {
     "name": "stderr",
     "output_type": "stream",
     "text": [
      "\n",
      "Bad key \"text.kerning_factor\" on line 4 in\n",
      "C:\\Users\\ocuri\\anaconda3\\envs\\pyvizenv\\lib\\site-packages\\matplotlib\\mpl-data\\stylelib\\_classic_test_patch.mplstyle.\n",
      "You probably need to get an updated matplotlibrc file from\n",
      "http://github.com/matplotlib/matplotlib/blob/master/matplotlibrc.template\n",
      "or from the matplotlib source distribution\n"
     ]
    }
   ],
   "source": [
    "import numpy as np\n",
    "import pandas as pd\n",
    "from pathlib import Path\n",
    "%matplotlib inline"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Regression Analysis: Seasonal Effects with Sklearn Linear Regression\n",
    "In this notebook, you will build a SKLearn linear regression model to predict Yen futures (\"settle\") returns with *lagged* Yen futures returns. "
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Futures contract on the Yen-dollar exchange rate:\n",
    "# This is the continuous chain of the futures contracts that are 1 month to expiration\n",
    "yen_futures = pd.read_csv(\n",
    "    Path(\"yen.csv\"), index_col=\"Date\", infer_datetime_format=True, parse_dates=True\n",
    ")\n",
    "yen_futures.head()"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 3,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Open</th>\n",
       "      <th>High</th>\n",
       "      <th>Low</th>\n",
       "      <th>Last</th>\n",
       "      <th>Change</th>\n",
       "      <th>Settle</th>\n",
       "      <th>Volume</th>\n",
       "      <th>Previous Day Open Interest</th>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Date</th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>1990-01-02</th>\n",
       "      <td>6954.0</td>\n",
       "      <td>6954.0</td>\n",
       "      <td>6835.0</td>\n",
       "      <td>6847.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>6847.0</td>\n",
       "      <td>48336.0</td>\n",
       "      <td>51473.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1990-01-03</th>\n",
       "      <td>6877.0</td>\n",
       "      <td>6910.0</td>\n",
       "      <td>6865.0</td>\n",
       "      <td>6887.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>6887.0</td>\n",
       "      <td>38206.0</td>\n",
       "      <td>53860.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1990-01-04</th>\n",
       "      <td>6937.0</td>\n",
       "      <td>7030.0</td>\n",
       "      <td>6924.0</td>\n",
       "      <td>7008.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>7008.0</td>\n",
       "      <td>49649.0</td>\n",
       "      <td>55699.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1990-01-05</th>\n",
       "      <td>6952.0</td>\n",
       "      <td>6985.0</td>\n",
       "      <td>6942.0</td>\n",
       "      <td>6950.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>6950.0</td>\n",
       "      <td>29944.0</td>\n",
       "      <td>53111.0</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>1990-01-08</th>\n",
       "      <td>6936.0</td>\n",
       "      <td>6972.0</td>\n",
       "      <td>6936.0</td>\n",
       "      <td>6959.0</td>\n",
       "      <td>NaN</td>\n",
       "      <td>6959.0</td>\n",
       "      <td>19763.0</td>\n",
       "      <td>52072.0</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "              Open    High     Low    Last  Change  Settle   Volume  \\\n",
       "Date                                                                  \n",
       "1990-01-02  6954.0  6954.0  6835.0  6847.0     NaN  6847.0  48336.0   \n",
       "1990-01-03  6877.0  6910.0  6865.0  6887.0     NaN  6887.0  38206.0   \n",
       "1990-01-04  6937.0  7030.0  6924.0  7008.0     NaN  7008.0  49649.0   \n",
       "1990-01-05  6952.0  6985.0  6942.0  6950.0     NaN  6950.0  29944.0   \n",
       "1990-01-08  6936.0  6972.0  6936.0  6959.0     NaN  6959.0  19763.0   \n",
       "\n",
       "            Previous Day Open Interest  \n",
       "Date                                    \n",
       "1990-01-02                     51473.0  \n",
       "1990-01-03                     53860.0  \n",
       "1990-01-04                     55699.0  \n",
       "1990-01-05                     53111.0  \n",
       "1990-01-08                     52072.0  "
      ]
     },
     "execution_count": 3,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Trim the dataset to begin on January 1st, 1990\n",
    "yen_futures = yen_futures.loc[\"1990-01-01\":, :]\n",
    "yen_futures.head()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Data Preparation"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Returns"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 4,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Create a series using \"Settle\" price percentage returns, drop any nan\"s, and check the results:\n",
    "# (Make sure to multiply the pct_change() results by 100)\n",
    "# In this case, you may have to replace inf, -inf values with np.nan\"s\n",
    "yen_futures['returns'] = (yen_futures[[\"Settle\"]].pct_change() * 100)\n",
    "yen_futures['returns'] = yen_futures['returns'].replace(-np.inf, np.nan).dropna()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Lagged Returns "
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 5,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Open</th>\n",
       "      <th>High</th>\n",
       "      <th>Low</th>\n",
       "      <th>Last</th>\n",
       "      <th>Change</th>\n",
       "      <th>Settle</th>\n",
       "      <th>Volume</th>\n",
       "      <th>Previous Day Open Interest</th>\n",
       "      <th>returns</th>\n",
       "      <th>Lagged_Return</th>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Date</th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>2014-02-18</th>\n",
       "      <td>9831.0</td>\n",
       "      <td>9865.0</td>\n",
       "      <td>9734.0</td>\n",
       "      <td>9775.0</td>\n",
       "      <td>42.0</td>\n",
       "      <td>9775.0</td>\n",
       "      <td>203495.0</td>\n",
       "      <td>196924.0</td>\n",
       "      <td>-0.427829</td>\n",
       "      <td>0.409123</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-19</th>\n",
       "      <td>9768.0</td>\n",
       "      <td>9825.0</td>\n",
       "      <td>9760.0</td>\n",
       "      <td>9773.0</td>\n",
       "      <td>2.0</td>\n",
       "      <td>9773.0</td>\n",
       "      <td>129508.0</td>\n",
       "      <td>197197.0</td>\n",
       "      <td>-0.020460</td>\n",
       "      <td>-0.427829</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-20</th>\n",
       "      <td>9774.0</td>\n",
       "      <td>9837.0</td>\n",
       "      <td>9765.0</td>\n",
       "      <td>9775.0</td>\n",
       "      <td>2.0</td>\n",
       "      <td>9775.0</td>\n",
       "      <td>160202.0</td>\n",
       "      <td>198280.0</td>\n",
       "      <td>0.020465</td>\n",
       "      <td>-0.020460</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-21</th>\n",
       "      <td>9772.0</td>\n",
       "      <td>9776.0</td>\n",
       "      <td>9725.0</td>\n",
       "      <td>9758.0</td>\n",
       "      <td>20.0</td>\n",
       "      <td>9755.0</td>\n",
       "      <td>103091.0</td>\n",
       "      <td>202990.0</td>\n",
       "      <td>-0.204604</td>\n",
       "      <td>0.020465</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-24</th>\n",
       "      <td>9752.0</td>\n",
       "      <td>9789.0</td>\n",
       "      <td>9740.0</td>\n",
       "      <td>9757.0</td>\n",
       "      <td>2.0</td>\n",
       "      <td>9757.0</td>\n",
       "      <td>90654.0</td>\n",
       "      <td>203114.0</td>\n",
       "      <td>0.020502</td>\n",
       "      <td>-0.204604</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "              Open    High     Low    Last  Change  Settle    Volume  \\\n",
       "Date                                                                   \n",
       "2014-02-18  9831.0  9865.0  9734.0  9775.0    42.0  9775.0  203495.0   \n",
       "2014-02-19  9768.0  9825.0  9760.0  9773.0     2.0  9773.0  129508.0   \n",
       "2014-02-20  9774.0  9837.0  9765.0  9775.0     2.0  9775.0  160202.0   \n",
       "2014-02-21  9772.0  9776.0  9725.0  9758.0    20.0  9755.0  103091.0   \n",
       "2014-02-24  9752.0  9789.0  9740.0  9757.0     2.0  9757.0   90654.0   \n",
       "\n",
       "            Previous Day Open Interest   returns  Lagged_Return  \n",
       "Date                                                             \n",
       "2014-02-18                    196924.0 -0.427829       0.409123  \n",
       "2014-02-19                    197197.0 -0.020460      -0.427829  \n",
       "2014-02-20                    198280.0  0.020465      -0.020460  \n",
       "2014-02-21                    202990.0 -0.204604       0.020465  \n",
       "2014-02-24                    203114.0  0.020502      -0.204604  "
      ]
     },
     "execution_count": 5,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Create a lagged return using the shift function\n",
    "yen_futures['Lagged_Return'] = yen_futures.returns.shift()\n",
    "yen2 = yen_futures.dropna()\n",
    "yen2.head()\n"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "### Train Test Split"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 10,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Create a train/test split for the data using 2018-2019 for testing and the rest for training\n",
    "train = yen2[:'2017']\n",
    "test = yen2['2018':]"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 11,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Create four dataframes:\n",
    "# X_train (training set using just the independent variables), X_test (test set of of just the independent variables)\n",
    "# Y_train (training set using just the \"y\" variable, i.e., \"Futures Return\"), Y_test (test set of just the \"y\" variable):\n",
    "X_train = train[\"Lagged_Return\"].to_frame()\n",
    "X_test = test[\"Lagged_Return\"].to_frame()\n",
    "y_train = train[\"returns\"]\n",
    "y_test = test[\"returns\"]"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 12,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>Lagged_Return</th>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Date</th>\n",
       "      <th></th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>2014-02-18</th>\n",
       "      <td>0.409123</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-19</th>\n",
       "      <td>-0.427829</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-20</th>\n",
       "      <td>-0.020460</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-21</th>\n",
       "      <td>0.020465</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2014-02-24</th>\n",
       "      <td>-0.204604</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "            Lagged_Return\n",
       "Date                     \n",
       "2014-02-18       0.409123\n",
       "2014-02-19      -0.427829\n",
       "2014-02-20      -0.020460\n",
       "2014-02-21       0.020465\n",
       "2014-02-24      -0.204604"
      ]
     },
     "execution_count": 12,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "X_train.head()"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Linear Regression Model"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 14,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "LinearRegression()"
      ]
     },
     "execution_count": 14,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Create a Linear Regression model and fit it to the training data\n",
    "from sklearn.linear_model import LinearRegression\n",
    "\n",
    "# Fit a SKLearn linear regression using just the training set (X_train, Y_train):\n",
    "model = LinearRegression()\n",
    "model.fit(X_train, y_train)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Make predictions using the Testing Data\n",
    "\n",
    "Note: We want to evaluate the model using data that it has never seen before, in this case: X_test."
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 15,
   "metadata": {},
   "outputs": [],
   "source": [
    "# Make a prediction of \"y\" values using just the test dataset\n",
    "predictions = model.predict(X_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 17,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/html": [
       "<div>\n",
       "<style scoped>\n",
       "    .dataframe tbody tr th:only-of-type {\n",
       "        vertical-align: middle;\n",
       "    }\n",
       "\n",
       "    .dataframe tbody tr th {\n",
       "        vertical-align: top;\n",
       "    }\n",
       "\n",
       "    .dataframe thead th {\n",
       "        text-align: right;\n",
       "    }\n",
       "</style>\n",
       "<table border=\"1\" class=\"dataframe\">\n",
       "  <thead>\n",
       "    <tr style=\"text-align: right;\">\n",
       "      <th></th>\n",
       "      <th>returns</th>\n",
       "      <th>Predicted Return</th>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>Date</th>\n",
       "      <th></th>\n",
       "      <th></th>\n",
       "    </tr>\n",
       "  </thead>\n",
       "  <tbody>\n",
       "    <tr>\n",
       "      <th>2018-01-02</th>\n",
       "      <td>0.297285</td>\n",
       "      <td>-0.009599</td>\n",
       "    </tr>\n",
       "    <tr>\n",
       "      <th>2018-01-03</th>\n",
       "      <td>-0.240479</td>\n",
       "      <td>-0.010033</td>\n",
       "    </tr>\n",
       "  </tbody>\n",
       "</table>\n",
       "</div>"
      ],
      "text/plain": [
       "             returns  Predicted Return\n",
       "Date                                  \n",
       "2018-01-02  0.297285         -0.009599\n",
       "2018-01-03 -0.240479         -0.010033"
      ]
     },
     "execution_count": 17,
     "metadata": {},
     "output_type": "execute_result"
    }
   ],
   "source": [
    "# Assemble actual y data (Y_test) with predicted y data (from just above) into two columns in a dataframe:\n",
    "Results = y_test.to_frame()\n",
    "Results[\"Predicted Return\"] = predictions\n",
    "Results.head(2)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 18,
   "metadata": {},
   "outputs": [
    {
     "data": {
      "text/plain": [
       "array([<matplotlib.axes._subplots.AxesSubplot object at 0x000001F69E0B46C8>,\n",
       "       <matplotlib.axes._subplots.AxesSubplot object at 0x000001F69E374F48>],\n",
       "      dtype=object)"
      ]
     },
     "execution_count": 18,
     "metadata": {},
     "output_type": "execute_result"
    },
    {
     "data": {
      "image/png": "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\n",
      "text/plain": [
       "<Figure size 432x288 with 2 Axes>"
      ]
     },
     "metadata": {
      "needs_background": "light"
     },
     "output_type": "display_data"
    }
   ],
   "source": [
    "# Plot the first 20 predictions vs the true values\n",
    "Results[:20].plot(subplots=True)"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Out-of-Sample Performance\n",
    "\n",
    "Evaluate the model using \"out-of-sample\" data (X_test and y_test)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 20,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "Out-of-Sample Root Mean Squared Error (RMSE): 0.41545437184712763\n"
     ]
    }
   ],
   "source": [
    "from sklearn.metrics import mean_squared_error\n",
    "# Calculate the mean_squared_error (MSE) on actual versus predicted test \"y\" \n",
    "mse = mean_squared_error(\n",
    "    Results[\"returns\"],\n",
    "    Results[\"Predicted Return\"]\n",
    ")\n",
    "\n",
    "# Using that mean-squared-error, calculate the root-mean-squared error (RMSE):\n",
    "rmse = np.sqrt(mse)\n",
    "print(f\"Out-of-Sample Root Mean Squared Error (RMSE): {rmse}\")"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# In-Sample Performance\n",
    "\n",
    "Evaluate the model using in-sample data (X_train and y_train)"
   ]
  },
  {
   "cell_type": "code",
   "execution_count": 22,
   "metadata": {},
   "outputs": [
    {
     "name": "stdout",
     "output_type": "stream",
     "text": [
      "In-sample Root Mean Squared Error (RMSE): 0.5962037920929946\n"
     ]
    }
   ],
   "source": [
    "# Construct a dataframe using just the \"y\" training data:\n",
    "in_sample_results = y_train.to_frame()\n",
    "\n",
    "# Add a column of \"in-sample\" predictions to that dataframe:  \n",
    "in_sample_results[\"In-sample Predictions\"] = model.predict(X_train)\n",
    "\n",
    "# Calculate in-sample mean_squared_error (for comparison to out-of-sample)\n",
    "in_sample_mse = mean_squared_error(\n",
    "    in_sample_results[\"returns\"],\n",
    "    in_sample_results[\"In-sample Predictions\"]\n",
    ")\n",
    "\n",
    "# Calculate in-sample root mean_squared_error (for comparison to out-of-sample)\n",
    "in_sample_rmse = np.sqrt(in_sample_mse)\n",
    "print(f\"In-sample Root Mean Squared Error (RMSE): {in_sample_rmse}\")"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "# Conclusions"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "YOUR CONCLUSIONS HERE!"
   ]
  },
  {
   "cell_type": "markdown",
   "metadata": {},
   "source": [
    "Lagged Yen futures returns is not a good predictor for Yen futures (\"settle\") returns and . The predictions using the testing data shows great difference between predicted returns and actual returns.  In addition, the out-of-smaple RMSE is 0.4154 compared to the In-sample RMSE of 0.5962.  Using the time-series method or using a training dataset that is 1 year before the testing dataset performed even worse than the poor predictability of using the testing set itself."
   ]
  }
 ],
 "metadata": {
  "file_extension": ".py",
  "kernelspec": {
   "display_name": "Python [conda env:pyvizenv] *",
   "language": "python",
   "name": "conda-env-pyvizenv-py"
  },
  "language_info": {
   "codemirror_mode": {
    "name": "ipython",
    "version": 3
   },
   "file_extension": ".py",
   "mimetype": "text/x-python",
   "name": "python",
   "nbconvert_exporter": "python",
   "pygments_lexer": "ipython3",
   "version": "3.7.9"
  },
  "mimetype": "text/x-python",
  "name": "python",
  "npconvert_exporter": "python",
  "pygments_lexer": "ipython3",
  "version": 3
 },
 "nbformat": 4,
 "nbformat_minor": 4
}
