import os
from google.oauth2.credentials import Credentials
from google_auth_oauthlib.flow import InstalledAppFlow
from googleapiclient.discovery import build

# Set up YouTube API credentials
SCOPES = ["https://www.googleapis.com/auth/youtube.force-ssl"]
API_SERVICE_NAME = "youtube"
API_VERSION = "v3"

def authenticate():
    credentials = None

    if os.path.exists("token.json"):
        credentials = Credentials.from_authorized_user_file("token.json")

    if not credentials or not credentials.valid:
        if credentials and credentials.expired and credentials.refresh_token:
            credentials.refresh(Request())
        else:
            flow = InstalledAppFlow.from_client_secrets_file(
                "credentials.json", SCOPES
            )
            credentials = flow.run_local_server(port=0)

        with open("token.json", "w") as token:
            token.write(credentials.to_json())

    return credentials

def search_random_video(api_key, query):
    youtube = build(API_SERVICE_NAME, API_VERSION, developerKey=api_key)

    search_response = youtube.search().list(
        q=query,
        type="video",
        part="id,snippet",
        maxResults=10,
    ).execute()

    videos = []
    for search_result in search_response.get("items", []):
        videos.append(
            {
                "title": search_result["snippet"]["title"],
                "video_id": search_result["id"]["videoId"],
            }
        )

    return videos

def post_random_comment(api_key, video_id, comments):
    youtube = build(API_SERVICE_NAME, API_VERSION, developerKey=api_key)

    selected_comment = random.choice(comments)

    comment_response = youtube.commentThreads().insert(
        part="snippet",
        body={
            "snippet": {
                "videoId": video_id,
                "topLevelComment": {
                    "snippet": {
                        "textOriginal": selected_comment,
                    }
                },
            }
        },
    ).execute()

    print(f"Comment posted: {selected_comment}")
    return comment_response

def main():
    api_key = "YOUR_YOUTUBE_API_KEY"
    query = "YOUR_SEARCH_QUERY"
    comments = [
        "Great video!",
        "Awesome content!",
        "Interesting video!",
        "Keep it up!",
    ]

    credentials = authenticate()

    if credentials:
        os.environ["OAUTHLIB_INSECURE_TRANSPORT"] = "1"
        video_results = search_random_video(api_key, query)

        if video_results:
            selected_video = random.choice(video_results)
            video_id = selected_video["video_id"]

            post_random_comment(api_key, video_id, comments)

if __name__ == "__main__":
    main()
