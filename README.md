# Recommendation Algorithm Ladder

Five recommendation algorithms, from simplest to a Transformer, built on the same
dataset with the same evaluation

## The ladder

| Step | Model | Family | Learns from |
|---|---|---|---|
| 1 | Popularity | Baseline | Item counts only |
| 2 | Item-kNN | Collaborative filtering (neighbourhood) | Which items are watched together |
| 3 | MF-BPR | Collaborative filtering (matrix factorization) | A learned vector per user and per item |
| 4 | Two-tower | Neural retrieval (hybrid) | IDs + demographics + genres + recent history |
| 5 | SASRec | Sequential (Transformer) | The order of what you watched |

## Setup

- **Dataset:** MovieLens-1M (ratings, movies and users files)
- **Feedback:** implicit. Every rating counts as "the user watched this movie"; star values are ignored.
- **Filtering:** keep users and movies with at least 5 ratings each
- **After filtering:** 6,040 users, 3,416 movies, 987,531 training interactions
- **IDs:** users numbered from 0, movies from 1 (ID 0 is reserved for padding)
- **Split:** leave-last-out, by time, per user.
  - Last movie: test
  - Second-to-last: validation (for tuning and early stopping)
  - Everything earlier: train
- **Evaluation:** full ranking. Every movie is scored for every user, movies already
  watched are hidden, and we check where the held-out test movie lands.
- **Metrics:** Recall@K and NDCG@K

## Results

| Model | Recall@10 | NDCG@10 | Recall@20 | NDCG@20 |
|---|---|---|---|---|
| Popularity | 0.0369 | 0.0180 | 0.0684 | 0.0259 |
| Item-kNN (k = 5) | 0.0818 | 0.0402 | 0.1328 | 0.0530 |
| MF-BPR | - | - | - | - |
| Two-tower | - | - | - | - |
| SASRec | - | - | - | - |

## Models
### 1. Popularity 
- Every user gets the same list of movie rankings: movies ordered by how many training users watched them, minus the ones that user has already seen.
- The validation split isn't used by Popularity, since there is nothing to tune. It starts to matter with Item-kNN (choosing k) and MF (early stopping)
- Recall@20 is 6.8%, so for about 7 users in 100 the next movie they watched was in their top 20.
- A random list of 20 out of about 3,400 movies would hit about 0.6% of the time, so Popularity is roughly 11 times better than random.
- NDCG@20 (2.6%) is well below Recall@20, meaning that when it hits, the movie tends to sit lower in the top 20 rather than at the very top.
- Popularity is a surprisingly strong floor for a model that learns nothing. Most users' next movie isn't one with high ratings

## 2. Item-kNN
- Different users get different recommendations: if user A watches movies 1 and 2, and other users who watch movie 1 and 2 also watch movie 3, then movie 3 is given a higher score for user A
- In Item-kNN, k is the number of neighbours each movie is allowed to keep. A movie's neighbours are the other movies most similar to it
- Small k: only the strongest links count. Less noise, but each movie recommends only a few others. Large k: lots of weak links count, more noise 
- k was tuned on the validation set over [4, 5, 10, 20, 30, 50, 100, 150, 200]. The best was k = 5, so only each movie's few strongest neighbours carry signal.
- Item-kNN roughly doubles Popularity's scores on every metric.