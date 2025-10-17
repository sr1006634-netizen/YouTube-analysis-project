# YouTube-analysis-project
import os import pandas as pd from googleapiclient.discovery import build

-----------------------------

Step 1: Set up YouTube API

-----------------------------

API_KEY = 'YOUR_YOUTUBE_API_KEY' YOUTUBE_API_SERVICE_NAME = 'youtube' YOUTUBE_API_VERSION = 'v3'

youtube = build(YOUTUBE_API_SERVICE_NAME, YOUTUBE_API_VERSION, developerKey=API_KEY)

-----------------------------

Step 2: Define function to fetch video data

-----------------------------

def get_channel_videos(channel_id, max_results=50): # Get uploads playlist ID res = youtube.channels().list(part='contentDetails', id=channel_id).execute() playlist_id = res['items'][0]['contentDetails']['relatedPlaylists']['uploads']

videos = []
next_page_token = None

while len(videos) < max_results:
    pl_request = youtube.playlistItems().list(
        part='snippet',
        playlistId=playlist_id,
        maxResults=min(max_results-len(videos),50),
        pageToken=next_page_token
    )
    pl_response = pl_request.execute()
    
    for item in pl_response['items']:
        video_id = item['snippet']['resourceId']['videoId']
        title = item['snippet']['title']
        videos.append({'video_id': video_id, 'title': title})
    
    next_page_token = pl_response.get('nextPageToken')
    if not next_page_token:
        break

return pd.DataFrame(videos)

-----------------------------

Step 3: Fetch video stats

-----------------------------

def get_video_stats(video_ids): stats = [] for i in range(0, len(video_ids), 50):  # API allows max 50 IDs per request request = youtube.videos().list( part='statistics,snippet', id=','.join(video_ids[i:i+50]) ) response = request.execute()

for item in response['items']:
        stats.append({
            'video_id': item['id'],
            'title': item['snippet']['title'],
            'publishedAt': item['snippet']['publishedAt'],
            'views': int(item['statistics'].get('viewCount', 0)),
            'likes': int(item['statistics'].get('likeCount', 0)),
            'comments': int(item['statistics'].get('commentCount', 0))
        })
return pd.DataFrame(stats)

-----------------------------

Step 4: Example usage

-----------------------------

channel_id = 'UC_x5XG1OV2P6uZZ5FSM9Ttw'  # Example: Google Developers channel videos_df = get_channel_videos(channel_id, max_results=20) stats_df = get_video_stats(videos_df['video_id'].tolist())

-----------------------------

Step 5: Save results

-----------------------------

stats_df.to_csv('youtube_channel_stats.csv', index=False) print("Saved video stats to youtube_channel_stats.csv")

