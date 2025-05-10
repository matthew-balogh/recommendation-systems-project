# Recommendation Systems Project

In this project, the goal is to implement a recommender system for a company in the field of e-commerce. For this scenario, the `Retailrocket E-commerce` [dataset from Kaggle](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset) is used.

The project objective is to experiment with various types of collaborative filtering, from simple and primitive ones, to exploring how matrix factorization could play a role in arriving at a better or more efficient model. Besides this objective, the implementation is driven by the business needs.

## Methodology

1.  Business needs

2.  Requirement details

3.  Model planning

4.  Model construction

5.  Evaluation

    1.  Recommendations

    2.  Verification

    3.  Interpretation

6.  Explanation

7.  Summary of different models

## Business Needs

**Home Page recommendations\
**When a user opens the home page of the online store, they should see relevant items as recommendations.

-   **Popular items**\
    The popularity can be measured from more angles, for example, total aggregated purchase count or average purchase count of an item.

-   **Items purchased by similar users**\
    The relevancy is based on the entire purchase history of the given user, for which similar users are selected and their most-relevant items are recommended for the user.

## Requirement Details

| Business need                    | In-app title                 | Expected behavior                                                                                                                                                         | Model name                             |
| -------------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| Popular item recommendation      | All-time Favorites           | The N most-frequently purchased items are recommended.                                                                                                                    | `PopularItemRecommender`               |
| Popular item recommendation      | Others are Coming Back for   | The N items with the largest mean purchase frequency are recommended.                                                                                                     | `MeanPopularItemRecommender`           |
| Items purchased by similar users | We think You will Love these | Based on the entire purchase history of the given user, similar K users are selected and N items from their purchases (not yet bought by the given user) are recommended. | `UserBasedCollabFilterItemRecommender` |

## Model Planning

| Model name                             | User elements              | Measurement                                                | Similarity | Filtering            | Selection   |
| -------------------------------------- | -------------------------- | ---------------------------------------------------------- | ---------- | -------------------- | ----------- |
| `PopularItemRecommender`               | \-                         | Purchase frequency, item-wise                              | \-         | \-                   | Top N items |
| `MeanPopularItemRecommender`           | \-                         | Mean purchase frequency, item-wise, after grouping by user | \-         | \-                   | Top N items |
| `UserBasedCollabFilterItemRecommender` | Previously purchased items | \-                                                         | \-         | K-most similar users | Top N items |

## Model Implementations

### Model 1.3. User-based Collaborative Filtering

The model is called as `UserBasedCollabFilterRecommender` and it takes the the pivot and similarity matrices as inputs on initialization. Calling the `.fit(userid)` method, the model is fitted for the given user (visitor). In the fitting phase, records of the pivot and similarity matrices related to the given user is selected and those items that the user has already purchased are flagged. Calling the `.recommend(k, n)` method, the model makes the recommendation for the fitted user, where `k` denotes the maximum number of similar users to be considered, and `n` denotes the maximum items to be retrieved as recommendations. The model makes sure that the given user is not included in the similar user collection and that items the user already purchased are not recommended and do not waste the result space defined by `k` and `n`.

After finding the similar users, the items associated with these users' purchases are retrieved in an ordered fashion: the more-frequently the item is bought the better its position is in this list.

The model provides helper functions to access fitting data and recommendation data by `.get_fit_memory()` and `.get_recommend_memory()`.


## Recommendations

### Model 1.1

![](./img/model_1a_recommendations.png)

### Model 1.2
![](./img/model_1b_recommendations.png)

## Verification

Verifications are issued after recommendations.

### Model 1.3. User-based Collaborative Filtering

`UserBasedCFRVerifier` ensures that the recommendations align with the requirements of `UserBasedCollabFilterRecommender`. Calling `run(designated_visitor_id)`, the following verifications are made:

* user in recommndation matches the input
* items flagged as already interacted with by the user are indeed in the original transactional dataset
* the number of users similar to the given user is at most $K$ and the given user is not in the collection
* the number of items recommended is at most $N$ and the items already interacted with by the user is not in the collection

## Interpretation and Explanation

### Model 1.1

![](./img/model_1a_explanation.png)

### Model 1.2
![](./img/model_1b_explanation.png)

## Literature Review