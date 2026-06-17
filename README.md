# Fakebook Recommendation System

## Overview

The Fakebook Recommendation System is responsible for generating personalized **For You Feed** content for each user.

The recommendation engine is designed in two phases:

* **Phase 1:** Rule-Based Multi-Signal Recommendation
* **Phase 2:** AI-Powered Semantic Recommendation

This phased approach allows the system to deliver personalized recommendations immediately while providing a clear migration path toward AI-driven content discovery.

---

# Phase 1: Multi-Signal Recommendation System

## Goal

Recommend posts based on user behavior, social relationships, content freshness, and engagement metrics.

## Architecture

```text
Candidate Collection
        ↓
Filtering
        ↓
Scoring Engine
        ↓
Ranking
        ↓
Top 50 Recommendations
```

---

## Candidate Collection

The system gathers posts from multiple sources:

### Following Posts

Posts created by users that the current user follows.

Conditions:

* Created within the last 3 days
* Ordered by newest first

### Popular Posts

Trending content across the platform.

Conditions:

* Created within the last 24 hours
* Ranked by likes and comments

### Recent Posts

Freshly published content.

Conditions:

* Created within the last hour
* Ordered by newest first

---

## Candidate Filtering

Before ranking, candidate posts are cleaned.

### Remove Previously Liked Posts

Posts already liked by the user are excluded.

Reason:

* Prevent repetitive recommendations
* Increase content diversity

### Deduplication

Posts collected from multiple sources are merged and deduplicated using Post ID.

---

## Ranking Signals

Each candidate post receives a score based on multiple recommendation signals.

### 1. Following Relationship Boost

If the user follows the author:

```text
Score = 100
Weight = 3
```

Purpose:

* Prioritize content from social connections.

---

### 2. Previous Interaction Boost

If the user previously liked content from the same author:

```text
Score = 50
Weight = 2
```

Purpose:

* Capture user preference toward specific creators.

---

### 3. Recency Decay Score

Newer posts receive higher scores.

Formula:

```text
10 × exp(-ageHours / 24)
```

Purpose:

* Keep feed fresh.

---

### 4. Engagement Score

Measures overall popularity.

Formula:

```text
Engagement = Likes + Comments × 2

Score = log(Engagement + 1) × 3
```

Purpose:

* Surface high-quality content.

---

### 5. Engagement Velocity Score

Measures how quickly engagement is growing.

Formula:

```text
Velocity = Engagement / AgeHours

Score = log(Velocity + 1) × 2
```

Purpose:

* Detect viral content.

---

## Final Score Formula

```text
Final Score =
(Following × 3)
+
(Interaction × 2)
+
Recency
+
Engagement
+
Velocity
```

Posts are sorted in descending order of Final Score.

---

## Recommendation Explainability

Every recommendation contains a score breakdown.

Example:

```text
Following (+300)
•
Liked Before (+100)
•
Recent (+8.4)
•
Popular (+6.1)
•
Viral (+3.2)
```

Benefits:

* Transparency
* Easier debugging
* Future analytics support

---

## Output

The recommendation engine returns:

```typescript
{
  posts: Post[],
  score: number,
  scoreBreakdown: string
}
```

Maximum:

```text
Top 50 posts
```

---

# Phase 2: AI-Powered Semantic Recommendation

## Goal

Move beyond social signals and recommend content based on semantic similarity and user interests.

Instead of asking:

> "Who posted this?"

The system asks:

> "Is this content relevant to the user's interests?"

---

## Architecture

```text
Post Creation
      ↓
Generate Embedding
      ↓
Store Vector
      ↓

User Interest Profile
      ↓
Generate User Embedding
      ↓

Vector Similarity Search
      ↓
Retrieve Similar Posts
      ↓
AI Recommendation Feed
```

---

## Embedding Generation

When a user creates a post:

```text
Title
+
Content
```

is converted into a vector embedding.

Example:

```text
"How to learn React effectively"
        ↓
[0.12, -0.45, 0.81, ...]
```

Possible models:

* Sentence Transformers
* BGE
* E5
* OpenAI Embeddings
* Instructor XL

---

## Vector Storage

Embeddings are stored inside a vector database.

Possible technologies:

* PostgreSQL + pgvector
* ChromaDB
* Pinecone
* Weaviate

Recommended for Fakebook:

```text
PostgreSQL + pgvector
```

Reason:

* Already using PostgreSQL
* Simpler deployment
* Lower operational cost

---

## User Interest Profile

User interests are inferred from:

### Likes

Posts liked by the user.

### Comments

Posts the user interacted with.

### Follows

Authors the user follows.

### View History (Future)

Posts viewed or opened.

---

## User Embedding

A user profile vector is generated from historical interactions.

Example:

```text
Liked:
- AI
- Machine Learning
- Data Science

↓

User Vector
```

---

## Similarity Search

Retrieve posts most similar to the user embedding.

Example query:

```sql
SELECT *
FROM posts
ORDER BY embedding <=> user_embedding
LIMIT 50;
```

Where:

```text
<=> = cosine distance
```

---

## Hybrid Recommendation

The final system combines:

### Traditional Signals

* Following
* Engagement
* Recency

### AI Signals

* Semantic similarity
* User interest matching

Formula:

```text
Final Score =
0.6 × Semantic Score
+
0.2 × Social Score
+
0.1 × Engagement Score
+
0.1 × Recency Score
```

---

# Future Improvements

* Personalized topic modeling
* Real-time recommendation updates
* Trending topic detection
* Collaborative filtering
* Graph-based recommendations
* Friend-of-friend content discovery
* Reinforcement learning ranking
* Multi-modal recommendation (text + image)

---

# Current Status

| Component            | Status     |
| -------------------- | ---------- |
| Candidate Collection | ✅          |
| Candidate Filtering  | ✅          |
| Ranking Engine       | ✅          |
| Explainability       | ✅          |
| Semantic Embeddings  | 🚧 Planned |
| Vector Database      | 🚧 Planned |
| Hybrid Ranking       | 🚧 Planned |

---

# Authors

Fakebook Development Team

Recommendation System v1.0
