# 🎬 Movie Recommender System

A content-based movie recommender system that suggests 5 similar movies based on your selection — built with Python, Scikit-learn, and Streamlit, powered by the TMDB dataset and API.

![Python](https://img.shields.io/badge/Python-3.8+-blue?style=for-the-badge&logo=python&logoColor=white)
![Scikit-learn](https://img.shields.io/badge/Scikit--learn-ML-orange?style=for-the-badge&logo=scikit-learn&logoColor=white)
![Streamlit](https://img.shields.io/badge/Streamlit-App-red?style=for-the-badge&logo=streamlit&logoColor=white)
![TMDB](https://img.shields.io/badge/TMDB-API-green?style=for-the-badge&logo=themoviedatabase&logoColor=white)

---

## 📌 Overview

Select any movie from a dropdown of **4800+ films** and the system instantly recommends **5 similar movies** with their posters — fetched live from the TMDB API.

This is a **Content-Based Filtering** recommender — it recommends movies based on the content of the movie itself (genres, keywords, cast, director, plot) rather than what other users watched.

---

## 🎯 Features

- Recommends **5 similar movies** for any selected film
- Displays **movie posters** fetched live from the TMDB API
- Built on a **4806 × 4806 cosine similarity matrix**
- Covers **4800+ movies** from the TMDB dataset
- Uses **genres, keywords, cast, director and overview** for similarity
- Clean **Streamlit web interface** with dropdown search

---

## 🖥️ Demo

Select a movie from the dropdown:

```
Selected: Inception
```

**Recommendations:**
```
1. The Dark Knight       2. Interstellar       3. The Prestige
4. Shutter Island        5. The Matrix
```
Each recommendation displays with its movie poster.

---

## 🗂️ Project Structure

```
movie-recommender-system/
│
├── app.py                        # Streamlit web application
├── notebook86c26b4f17.ipynb      # Data processing + similarity notebook
├── model/
│   ├── movie_list.pkl            # Saved DataFrame (movie_id, title, tags)
│   └── similarity.pkl            # Saved 4806×4806 cosine similarity matrix
├── README.md
```

---

## 🧠 How It Works

### Type of Recommender

| Type | How It Works | This Project |
|---|---|---|
| Collaborative Filtering | Based on what similar users liked | ❌ Not used |
| **Content-Based Filtering** | Based on the content/features of the movie | ✅ Used |

---

### Full Pipeline

```
Load TMDB Data (movies + credits)
         ↓
Merge on title
         ↓
Select useful columns
(movie_id, title, overview, genres, keywords, cast, crew)
         ↓
Parse JSON columns → extract names
         ↓
Keep top 3 cast + director only
         ↓
Remove spaces from names (JamesCameron, LeonardoDiCaprio)
         ↓
Combine all features into one 'tags' string per movie
         ↓
CountVectorizer (max 5000 features)
         ↓
Cosine Similarity Matrix (4806 × 4806)
         ↓
recommend() → top 5 similar movies
         ↓
Streamlit App + TMDB API posters
```

---

### 1. Data Loading & Merging
- Loaded two TMDB CSV files — `tmdb_5000_movies.csv` and `tmdb_5000_credits.csv`
- Merged on `title` column to combine movie details with cast/crew info
- Selected 7 key columns: `movie_id`, `title`, `overview`, `genres`, `keywords`, `cast`, `crew`

---

### 2. Feature Extraction
Raw columns contained JSON strings — parsed using `ast.literal_eval()`:

| Column | Raw | After Parsing |
|---|---|---|
| genres | `'[{"id":28,"name":"Action"},...]'` | `['Action', 'Adventure']` |
| keywords | `'[{"id":10,"name":"spy"},...]'` | `['spy', 'secret agent']` |
| cast | JSON of 50+ actors | Top 3 actors only |
| crew | JSON of all crew | Director only |

---

### 3. Building the Tags Column
Spaces removed from all multi-word names to preserve identity:
```
"Sam Worthington"   →  "SamWorthington"
"Science Fiction"   →  "ScienceFiction"
"James Cameron"     →  "JamesCameron"
```

All 5 features combined into one string per movie:
```python
tags = overview + genres + keywords + cast + crew
```

Example tags for Avatar:
```
"alien world future war JamesCameron SamWorthington ZoeSaldana
 SigourneyWeaver action adventure sciencefiction"
```

---

### 4. Vectorization
```python
cv = CountVectorizer(max_features=5000, stop_words='english')
vector = cv.fit_transform(new['tags']).toarray()
# Shape: (4806, 5000)
```
Each movie becomes a row of **5000 word-count numbers** — its numerical fingerprint.

---

### 5. Cosine Similarity
```python
from sklearn.metrics.pairwise import cosine_similarity
similarity = cosine_similarity(vector)
# Shape: (4806, 4806)
```

Every cell `similarity[i][j]` = how similar movie i is to movie j.
- Score = **1.0** → identical
- Score = **0.8+** → very similar
- Score = **0.0** → completely unrelated

Movies sharing the same director, cast, genres and keywords score highest.

---

### 6. Recommendation Logic
```python
def recommend(movie):
    index = movies[movies['title'] == movie].index[0]
    distances = sorted(list(enumerate(similarity[index])),
                       reverse=True, key=lambda x: x[1])
    for i in distances[1:6]:   # skip index 0 (the movie itself)
        print(movies.iloc[i[0]].title)
```

---

## 🚀 Getting Started

### Prerequisites
- Python 3.8 or above
- TMDB API key (free at [themoviedb.org](https://www.themoviedb.org/))

### Installation

```bash
# 1. Clone the repository
git clone https://github.com/your-username/movie-recommender-system.git
cd movie-recommender-system

# 2. Install dependencies
pip install streamlit scikit-learn pandas numpy requests

# 3. Run the app
streamlit run app.py
```

The app will open at `http://localhost:8501`

> **Note:** `model/movie_list.pkl` and `model/similarity.pkl` must exist. If missing, run the notebook end-to-end to regenerate them.

---

## 📦 Dependencies

```
streamlit
scikit-learn
pandas
numpy
requests
```

---

## 📁 Dataset

| Detail | Value |
|---|---|
| Source | TMDB 5000 Movie Dataset (Kaggle) |
| Movies file | tmdb_5000_movies.csv |
| Credits file | tmdb_5000_credits.csv |
| Total movies | 4803 |
| After cleaning | 4806 (post merge) |
| Features used | genres, keywords, cast (top 3), director, overview |

---

## 🔄 Prediction Flow in app.py

```python
# 1. User selects a movie from dropdown
selected_movie = st.selectbox("Type or select a movie", movie_list)

# 2. Fetch poster from TMDB API
def fetch_poster(movie_id):
    url = f"https://api.themoviedb.org/3/movie/{movie_id}?api_key=YOUR_KEY"
    data = requests.get(url).json()
    return "https://image.tmdb.org/t/p/w500/" + data['poster_path']

# 3. Find top 5 similar movies using cosine similarity
# 4. Display names + posters in 5 side-by-side columns
col1, col2, col3, col4, col5 = st.beta_columns(5)
```

---

## ⚠️ Limitations

- **No stemming** — "love" and "loving" treated as different words (intentional — preserves actor/director names)
- **No user history** — purely content based, does not learn from viewing habits
- **Poster dependency** — requires live TMDB API calls; posters won't load if API key expires
- **similarity.pkl is large** — ~50-100MB file may cause issues on free hosting platforms
- Dataset limited to movies available in TMDB up to 2017

---

## 🔮 Future Improvements

- [ ] Add Collaborative Filtering for hybrid recommendations
- [ ] Include movie ratings and popularity as additional features
- [ ] Add genre filter for more refined recommendations
- [ ] Show movie overview, ratings and release year alongside poster
- [ ] Deploy on Streamlit Cloud with compressed similarity matrix

---


---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

