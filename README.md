# data-privacy-final-project

## Problem Statement: 

Given a dataset containing numerous features of wine (fixed_acidity, volatile_acidity,...) I aim to predict wine quality, an integer, using a differentially private gradient descent algorithm. I will be using mean squared error (MSE) for the gradient descent algorithm, which is typically used when trying to predict a continuous value. 

---

## Technical Description: 

First, the wine CSV files were read into pandas dataframes. Then I created an X variable, which held all rows with the predictors of the wine quality, and the Y variable, which held all the rows with the label (wine quality.) I created my training and testing sets (X_train, y_train, X_test, y_test) by indexing my X and y variables, I set my training set to be 80% of my total data. After I created numerous functions to help with my implementation of noisy gradient descent. 

One function was created to compute the gradient of linear loss, gradient(). First I found the deriative of a MSE equation online (from: https://towardsdatascience.com/linear-regression-using-gradient-descent-97a6c8700931). Using that formula, I allow gradient() to take in three parameters, a feature vector (xi), the weights of the model (theta) and the actual Y corresponding to those features (yi). First, I make a prediction of Y by calculating the dot product of my theta and Xi. I return the updated gradient by subtracting pred from actual (pred-actual) and then multiplying that by the feature vectors, then I multiply that product by -2. So overall it looks like: -2 * (xi * (yi - y_pred)).

Two other functions aided in my creation of the noisy_gradient_descent(), they were gaussian_mech_vec() and L2_clip(). gaussian_mech_vec() is used in my function to add noise to the sum of my gradients. It takes in a vector (presumably gradients), sensitivity, epsilon, delta. gaussian_mech_vec() adds gaussian noise to each gradient in my vector of gradients, with the amount of noise being determined by sensitivity, epsilon and delta. L2_clip() takes in a clipping parameter (b) and a vector (v). First, L2_clip() finds the L2 norm of the given vector. Then it runs a simple test, if that norm is greater than the clipping parameter then we want to scale the vector. We can accomplish this by dividing the vector by its L2 norm and then multipying by clipping parameter b. This ensures the vector has at most a b value of 5. If the norm is less than the clipping parameter it leaves the vector unchanged, it has met our condition. Note: L2 norm cannot be negative, will always be between value 0-b.

noisy_gradient_descent() takes in three parameters, the amount of times to perform noisy gradient descent (iterations), two privacy parameters set by the user (epsilon, delta). Variables are set before the main functionality of noisy GD, these include: learning rate - which can help slow down learning so we don't jump too far past the optimal solution in our gradient descent calculation, theta - orginally the weights are set to zero, same amount as number of features in our feature vector (fixed_acidity, volatile_acidity,...), epislon_i and delta_i are the given privacy parameters set by the user divided by the number of iterations. Note: More iterations for a DP-gradient descent algorithm can make the model worse, since we have to use a smaller person for each iteration and so scale the noise goes up. We then peform the gradient descent iterations amount of times. Here is the steps inside are loop: 1. Compute the gradients per example in X_train using our gradient() function. 2. Call the L2_clip() function on our vector of calculated gradients 3. Take the sum of the gradients using np.sum 4. add noise to the sum of the gradients through the gaussian_mech_vec(), setting b to the L2_clip() clipping parameter, epsilon to epsilon_i and delta to delta_i. Then, we take the noisy sum of gradients and divide it by the length of the training set. Note: this breaks DP, but its probably fine to reveal, we can add noise to length of gradient to break it. Lastly, we set theta to be equal to theta minus learning rate times noisy gradient. At the end of the iterations we return our final theta. 

Two functions were used to evaluate how my test set did with my model, predict() and accuracy(). predict() takes in a feature vector (xi), and the weights of the model (theta) to predict a label (continuous value) of wine. accuracy() takes in the weights of the model (theta) and calls predict() on the entire test set and evaluates it by subtracting the predicted by the actual and squaring that result (MSE). 

---

## Description of results: 

I created two other files for baselines on my data. First, I ran gradient descent on my data by fitting it to a sci-kit learn LinearRegression() model and evaluted it using MSE. Then I ran my coded version of gradient descent without noise for another baseline. Lastly, as described above I implemented noisy gradient descent. Here are the benchmarks. Key note: I used the white wine dataset for benchmarking due to the higher number of examples for training and testing. All benchmarks used the same data with the same train/test split percentage. Also, epsilon was set to 1.0 and Delta to 1e-5 (0.00001) for gradient_descent() and noisy_gradient_descent().  It was found that the gradient_descent() converges at around 1800 iterations of GD with a LR of 0.000042. For the MSE score for noisy_gradient_descent(), I also found the optimal solution converged at around 1800 iterations, so I ran noisy_gd 10 different times with 1800 iterations and averaged the results to find the mse score.

LinearRegression() with scikit-learn MSE score: 0.5072104636729868
gradient_descent(): 0.5677132593477637
noisy_gradient_descent(): 4.153564548955102

Overall, the results with noisy gradient descent were less successful. Seeing as the ranges of the quality range from 3-7 for white wine, having a MSE of 4.15 is not helpful. I attempted using other DP variants that provide tighter bounds on privacy costs including zero-concentrated differential privacy (zCDP) and Rényi differential privacy (RDP). First, when using zCDP with a rho value=0.1, my MSE was similar, scoring a 4.14. Given a rho value of 0.1 and a delta value of 1e-5, it represents an epsilon value approximately of 2.2. So even when increasing my privacy budget and using a variant of DP that provides a better bound on privacy, it did not score better. Similar results were found when using RDP. 

However, I believe with parameter optimization we could see an increase in the results (decrease in MSE). 

## Citations
 P. Cortez, A. Cerdeira, F. Almeida, T. Matos and J. Reis. 
  Modeling wine preferences by data mining from physicochemical properties.
  In Decision Support Systems, Elsevier, 47(4):547-553. ISSN: 0167-9236.

  Menon, Adarsh. “Linear Regression Using Gradient Descent.” Medium, Towards Data Science, 19 Sept. 2018, towardsdatascience.com/linear-regression-using-gradient-descent-97a6c8700931. 