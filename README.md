# Recommendation Systems Project

In this project, the goal is to implement a recommender system for a company in the field of e-commerce. For this scenario, the `Retailrocket E-commerce` [dataset from Kaggle](https://www.kaggle.com/datasets/retailrocket/ecommerce-dataset) is used.

The project objective is to experiment with various types of collaborative filtering, from simple and primitive ones, to exploring how matrix factorization could play a role in arriving at a better or more efficient model. Besides this objective, the implementation is driven by the business needs.

## Methodology

1.  Business needs

2.  Requirement details

3.  Model planning

4.  Model construction

5.  Explanation

6.  Evaluation

7.  Summary of different models

## Business Needs

Item recommendations for visitors in an online store based on implicit feedback to provide better user experience and to boost sales.

**Home Page recommendations**\
When a user opens the home page of the online store, they should see relevant items as recommendations.

-   **Popular items**\
    The popularity can be measured from more angles, for example, total aggregated purchase count or average purchase count of an item.

-   **Items purchased by similar users**\
    The relevancy is based on the entire purchase history of the given user, for which similar users are selected and their most-relevant items are recommended for the user.

![Wireframe for the home page of the online store. Three recommendation panels are depicted: two global and one filtering approaches.](./img/home.png)

**Newsletter recommendations**\
Eye-catching recommendations are sent to the users' mail.

-   **Discovered patterns**

![](img/newsletter.png)

## Requirement Details

| Business need                    | In-app title                   | Expected behavior                                                                                                                                                                                                          | Model name                             |
| -------------------------------- | ------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------- |
| Popular item recommendation      | *All-time Favorites*           | The $N$ most-frequently purchased items are recommended.                                                                                                                                                                   | `PopularItemRecommender`               |
| Popular item recommendation      | *Others are Coming Back for*   | The $N$ items with the largest mean purchase frequency are recommended.                                                                                                                                                    | `MeanPopularItemRecommender`           |
| Items purchased by similar users | *We think You will Love these* | Based on the entire purchase history of the given user, similar $K$ users are selected and $N$ items from their purchases (not yet bought by the given user) are recommended.                                              | `UserBasedCollabFilterItemRecommender` |
| Discover hidden patterns         | *Already Picked for You*       | Based on the entire purchase history of all the users, hidden factors are discovered via *Matrix Factorization*. $N$ not yet purchased items with the top score implied by the factors are recommended for the given user. | `MatrixFactorizationItemRecommender`   |

## Model Planning

| Model name                             | User elements                                                          | Measurement                                                | Similarity                                                    | Filtering                                 | Selection     |
| -------------------------------------- | ---------------------------------------------------------------------- | ---------------------------------------------------------- | ------------------------------------------------------------- | ----------------------------------------- | ------------- |
| `PopularItemRecommender`               | \-                                                                     | Purchase frequency, item-wise                              | \-                                                            | \-                                        | Top $N$ items |
| `MeanPopularItemRecommender`           | \-                                                                     | Mean purchase frequency, item-wise, after grouping by user | \-                                                            | \-                                        | Top $N$ items |
| `UserBasedCollabFilterItemRecommender` | Previously purchased items                                             | Binary purchase flag, user-item pairwise                   | Cosine similarity, user-user pairwise                         | $K$-most similar users (threshold: $0.1$) | Top $N$ items |
| `MatrixFactorizationItemRecommender`   | Previously purchased items for all users (implicit feedback, weighted) | Discovered latent factors ($U$, $V$)                       | Dot product of latent factors of users and items ($U$, $V.T$) | \-                                        | Top $N$ items |

![](img/model_planning_levels.png)

## Model Implementations

### `PopularItemRecommender`

`PopularItemRecommender` takes the transactional dataset as input on initialization. Calling the `.recommend(n)` method, the model creates the recommendation where `n` denotes the maximum items to be retrieved as recommendations. It simply groups the dataset by `itemid` and counts the transactional events to select the top N items. As it appears, no fitting, no filtering takes place, therefore the recommendations will be the same for all users.

### `MeanPopularItemRecommender`

`MeanPopularItemRecommender` takes the transactional dataset as input on initialization. Calling the `.recommend(n)` method, the model creates the recommendation where `n` denotes the maximum items to be retrieved as recommendations. It simply groups the dataset by `userid` and `itemid` and counts the transactional events within these group then averages them to select the top N items. As it appears, no fitting, no filtering takes place, therefore the recommendations will be the same for all users.

### `UserBasedCollabFilterItemRecommender`

The model is called as `UserBasedCollabFilterItemRecommender` and it takes the the pivot and similarity matrices as inputs on initialization. Calling the `.fit(userid)` method, the model is fitted for the given user (visitor). In the fitting phase, records of the pivot and similarity matrices related to the given user is selected and those items that the user has already purchased are flagged. Calling the `.recommend(k, n)` method, the model makes the recommendation for the fitted user, where `k` denotes the maximum number of similar users to be considered, and `n` denotes the maximum items to be retrieved as recommendations. The model makes sure that the given user is not included in the similar user collection and that items the user already purchased are not recommended and do not waste the result space defined by `k` and `n`.

After finding the similar users, the items associated with these users' purchases are retrieved in an ordered fashion: the more-frequently the item is bought the better its position is in this list.

The model provides helper functions to access fitting data and recommendation data by `.get_fit_memory()` and `.get_recommend_memory()`.

### `MatrixFactorizationItemRecommender`

The model is called as `MatrixFactorizationItemRecommender` and it incorporates a Weighted Matrix Factorization (`WMF`) algorithm from `cornac`. Calling the `.experiment(...)` method, the model performs a `cornac.Experiment` with the given arguments.

## Recommendations

### `PopularItemRecommender`

![](img/model_1_call.png)

|             |        |        |        |      |        |
| ----------- | -----: | -----: | -----: | ---: | -----: |
| **Item id** | 461686 | 119736 | 213834 | 7943 | 312728 |

### `MeanPopularItemRecommender`

![](img/model_2_call.png)

|             |        |        |      |        |        |
| ----------- | -----: | -----: | ---: | -----: | -----: |
| **Item id** | 396042 | 224549 |  147 | 218612 | 347641 |

### `UserBasedCollabFilterItemRecommender`

![](img/model_3_call.png)

|             |       |        |        |       |       |
| ----------- | ----: | -----: | -----: | ----: | ----: |
| **Item id** | 10572 | 171878 | 218794 | 40630 | 32581 |


## Explaining the Results

### `PopularItemRecommender`

If we look at the purchase frequency of each recommended item and associate them with the histogram of purchase frequencies, we can see that the recommended items are in the right tail of the distribution, that is the recommended items are the most popular in terms of item-wise purchase frequency.

![](./img/model_1_explanation.png)

### `MeanPopularItemRecommender`

If we look at the average purchase frequency of each recommended item and associate them with the histogram of average purchase frequencies, we can see that the recommended items are in the right tail of the distribution, that is the recommended items are the most popular in terms of item-wise average purchase frequency.

![](./img/model_2_explanation.png)

### `UserBasedCollabFilterItemRecommender`

If we look at the first diagram below, we can see the $K$ users most similar to the designated one ,highlighted with green color. As the visual exhibits, the $K$ similarities indeed have the largest scores based on *Cosine* similarity.

In the second diagram below, we can see the $N$ recommended items and those ($\le K$) similar users that purchased any of these recommended items before. Black color indicates the absence, while white the presence of such interaction. We can see that each recommended item has been purchased by at least one of the similar users.

![](img/model_3_explanation_1.png)

![](img/model_3_explanation_2.png)

## Summary

|                                  | Business need                 | Brief description                                                         | Levels | Advantages                                                                               | Limitations                                                                                                                                                                                            |
| -------------------------------- | ----------------------------- | ------------------------------------------------------------------------- | ------ | ---------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| PopularItemsRecommender          | 1\. Home page recommendations | Recommends the most-frequently purchased items.                           | Global | It is easy to implement and to recommend items. It is applicable in the cold-start case. | It may be biased by differences in purchase scales across items or users. It does not make any filtering related to the actual user: same results for everyone.                                        |
| PopularItemsOnAverageRecommender |                               | Recommends the most-frequently purchased items yet based on a mean value. | Global | It is easy to implement and to recommend items. It is applicable in the cold-start case. | It may be biased by differences in purchase scales across items but the differences across users is now reduced. It does not make any filtering related to the actual user: same results for everyone. |