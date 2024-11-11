import requests
import pandas as pd
import nltk
from collections import Counter
from nltk.corpus import stopwords
from nltk.tokenize import word_tokenize

# Ensure you download stopwords
nltk.download('stop')
nltk.download('punkt')

BEARER_TOKEN = 'tweepy'  # Replace with your actual token
LIST_ID = 'cimaparadiso@daum.net'            # Replace with the Twitter List ID

def create_headers(bearer_token):
    return {"Authorization": f"Bearer {bearer_token}"}

def get_tweets_from_list(headers, list_id, max_results=100):
    url = f"https://api.twitter.com/2/lists/{list_id}/tweets"
    params = {
        "max_results": max_results,
        "tweet.fields": "text,created_at",
    }
    response = requests.get(url, headers=headers, params=params)
    if response.status_code != 200:
        raise Exception(f"Request returned an error: {response.status_code}, {response.text}")
    return response.json()['data']

def preprocess_text(text):
    stop_words = set(stopwords.words('english'))
    tokens = word_tokenize(text.lower())
    tokens = [word for word in tokens if word.isalpha() and word not in stop_words]
    return tokens

def analyze_trends(tweets):
    all_words = []
    for tweet in tweets:
        tokens = preprocess_text(tweet['text'])
        all_words.extend(tokens)
    word_freq = Counter(all_words)
    return word_freq.most_common(20)

def main():
    headers = create_headers(BEARER_TOKEN)
    tweets = get_tweets_from_list(headers, LIST_ID, max_results=100)
    trends = analyze_trends(tweets)
    
    df = pd.DataFrame(trends, columns=['Token', 'Frequency'])
    print("Top Trends:\n", df)

if __name__ == "__main__":
    main()

