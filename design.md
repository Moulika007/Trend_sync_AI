# PulseVibe AI - System Design

## Architecture Overview

### High-Level Architecture
```
┌─────────────────────────────────────────────────────────────┐
│                    React.js Frontend                         │
│     (Dashboard, Transcript Upload, Content Preview)         │
└─────────────────────────────────────────────────────────────┘
                              │
                              │ REST API
                              ▼
┌─────────────────────────────────────────────────────────────┐
│              Node.js + Express Backend                       │
│        (Request Handler, API Gateway, Auth)                  │
└─────────────────────────────────────────────────────────────┘
                              │
        ┌─────────────────────┼─────────────────────┐
        │                     │                     │
        ▼                     ▼                     ▼
┌───────────────┐   ┌─────────────────┐   ┌────────────────┐
│  Child Process│   │  Google Gemini  │   │    MongoDB     │
│  Python Script│   │      API        │   │   Database     │
└───────────────┘   └─────────────────┘   └────────────────┘
        │
        ▼
┌───────────────────────────────────────────────────────────┐
│              Python Trend Engine                           │
│  (Pytrends Library - Google Trends API Wrapper)           │
└───────────────────────────────────────────────────────────┘
```

## The PulseVibe Flow (Step-by-Step)

1. **User Input**: Creator uploads transcript via React dashboard
2. **Backend Processing**: Node.js receives request and validates input
3. **Trend Fetching**: Backend spawns Python child process
4. **Pytrends Execution**: Python script fetches India-specific trends
5. **JSON Response**: Python returns trending keywords as JSON
6. **AI Processing**: Node.js sends transcript + trends to Gemini API
7. **Content Generation**: Gemini creates platform-specific content
8. **Data Storage**: MongoDB stores content and trend snapshots
9. **Scheduling Analysis**: Algorithm analyzes interest-over-time data
10. **Recommendation**: System suggests optimal posting times
11. **User Review**: Creator reviews and edits generated content
12. **Publishing**: Content posted to selected platforms

## Core Components

### 1. React.js Frontend
**Responsibilities:**
- User authentication and dashboard
- Transcript upload interface
- Content preview and editing
- Platform selection (Instagram, Facebook, YouTube)
- Scheduling interface
- Analytics visualization

**Tech Stack:**
- React.js 18+ with Hooks
- React Router for navigation
- Axios for API calls
- TailwindCSS for styling
- Chart.js for analytics

**Key Pages:**
- Dashboard (overview, quick actions)
- Upload (transcript input, file upload)
- Generate (platform selection, trend preview)
- Editor (AI-generated content review)
- Schedule (calendar view, timing recommendations)
- Analytics (performance metrics, trend insights)

### 2. Node.js + Express Backend
**Responsibilities:**
- RESTful API endpoints
- User authentication (JWT)
- Request validation and sanitization
- Python script orchestration
- Gemini API integration
- Database operations
- Social media API integration

**Tech Stack:**
- Node.js 18+
- Express.js framework
- JWT for authentication
- Mongoose for MongoDB
- Child Process for Python execution
- Axios for external APIs

**Key Endpoints:**
```javascript
POST   /api/auth/register
POST   /api/auth/login
POST   /api/content/generate
GET    /api/trends/:keyword
POST   /api/schedule/create
GET    /api/analytics/performance
POST   /api/publish/:platform
```

### 3. Python Trend Engine
**Responsibilities:**
- Fetch real-time Google Trends data
- India-specific geo-targeting
- Interest-over-time analysis
- Related queries extraction
- Rising topics identification
- JSON output for Node.js consumption

**Tech Stack:**
- Python 3.10+
- Pytrends library
- JSON for data exchange
- Pandas for data processing

**Key Script: trend_fetcher.py**
```python
from pytrends.request import TrendReq
import json
import sys

def fetch_trends(keyword, region='IN'):
    pytrends = TrendReq(hl='en-IN', tz=330)
    pytrends.build_payload([keyword], geo=region, timeframe='now 7-d')
    
    # Interest over time
    interest = pytrends.interest_over_time()
    
    # Related queries
    related = pytrends.related_queries()
    
    # Trending searches
    trending = pytrends.trending_searches(pn='india')
    
    result = {
        'interest_over_time': interest.to_dict(),
        'related_queries': related,
        'trending_searches': trending.to_list()
    }
    
    print(json.dumps(result))

if __name__ == '__main__':
    keyword = sys.argv[1]
    region = sys.argv[2] if len(sys.argv) > 2 else 'IN'
    fetch_trends(keyword, region)
```

### 4. Google Gemini AI Integration
**Responsibilities:**
- Platform-specific content generation
- Sentiment-aware transformation
- Trending keyword injection
- Multi-language support
- Tone and style adaptation

**Tech Stack:**
- Google Gemini API (gemini-pro model)
- Prompt engineering templates
- Context management

**Platform-Specific Prompts:**
```javascript
const prompts = {
  instagram: `Act as a viral Instagram influencer targeting Indian millennials. 
              Create an engaging caption with emojis and trending hashtags.
              Trending keywords to include: {trends}
              Transcript: {transcript}`,
  
  youtube: `Act as a YouTube SEO expert. Create an optimized title and description 
            for Indian search queries with trending keywords.
            Trending keywords: {trends}
            Transcript: {transcript}`,
  
  facebook: `Act as a community manager. Create a post that encourages discussion 
             and engagement in Indian Facebook groups.
             Trending keywords: {trends}
             Transcript: {transcript}`
};
```

### 5. MongoDB Database
**Responsibilities:**
- User data storage
- Content history
- Trend snapshots
- Scheduled posts
- Analytics data

**Collections:**
```javascript
users: {
  _id, email, password_hash, name, created_at, preferences
}

content: {
  _id, user_id, transcript, generated_posts: {
    instagram, facebook, youtube
  }, trends_used, created_at, status
}

trends: {
  _id, keyword, region, interest_data, related_queries, 
  trending_searches, fetched_at
}

scheduled_posts: {
  _id, content_id, platform, scheduled_time, status, 
  posted_at, performance_metrics
}
```

## Technical Implementation Details

### A. Bridging Node.js and Python

**The Challenge:**
Python's Pytrends library is the most reliable way to access Google Trends data, but our backend is Node.js. We need seamless communication between the two.

**The Solution:**
Use Node.js `child_process` module to spawn Python scripts as background workers. The Python script outputs JSON to stdout, which Node.js captures and parses.

**Implementation:**

**Node.js Backend (server/services/trendService.js):**
```javascript
const { exec } = require('child_process');
const path = require('path');

class TrendService {
  async fetchTrends(keyword, region = 'IN') {
    return new Promise((resolve, reject) => {
      const scriptPath = path.join(__dirname, '../../python/trend_fetcher.py');
      const command = `python3 ${scriptPath} "${keyword}" "${region}"`;
      
      exec(command, { maxBuffer: 1024 * 1024 }, (error, stdout, stderr) => {
        if (error) {
          console.error('Python script error:', stderr);
          reject(new Error('Failed to fetch trends'));
          return;
        }
        
        try {
          const trends = JSON.parse(stdout);
          resolve(trends);
        } catch (parseError) {
          reject(new Error('Invalid JSON from Python script'));
        }
      });
    });
  }
  
  calculateOptimalTime(interestData) {
    // Calculate 3-day moving average
    const values = Object.values(interestData);
    const movingAvg = [];
    
    for (let i = 2; i < values.length; i++) {
      const avg = (values[i-2] + values[i-1] + values[i]) / 3;
      movingAvg.push(avg);
    }
    
    // Find positive slope (rising trend)
    for (let i = 1; i < movingAvg.length; i++) {
      if (movingAvg[i] > movingAvg[i-1]) {
        return {
          recommended: true,
          reason: 'Rising trend detected',
          postIn: '2-4 hours'
        };
      }
    }
    
    return {
      recommended: false,
      reason: 'No rising trend',
      postIn: 'Wait for trend spike'
    };
  }
}

module.exports = new TrendService();
```

**Python Script (python/trend_fetcher.py):**
```python
from pytrends.request import TrendReq
import json
import sys
from datetime import datetime

def fetch_trends(keyword, region='IN'):
    try:
        # Initialize Pytrends
        pytrends = TrendReq(hl='en-IN', tz=330)
        
        # Build payload
        pytrends.build_payload([keyword], geo=region, timeframe='now 7-d')
        
        # Interest over time
        interest_df = pytrends.interest_over_time()
        interest_dict = {}
        if not interest_df.empty:
            interest_dict = interest_df[keyword].to_dict()
            # Convert timestamps to strings
            interest_dict = {str(k): v for k, v in interest_dict.items()}
        
        # Related queries
        related = pytrends.related_queries()
        related_top = []
        related_rising = []
        
        if keyword in related and related[keyword]['top'] is not None:
            related_top = related[keyword]['top']['query'].tolist()[:10]
        if keyword in related and related[keyword]['rising'] is not None:
            related_rising = related[keyword]['rising']['query'].tolist()[:10]
        
        # Trending searches in India
        trending_df = pytrends.trending_searches(pn='india')
        trending_list = trending_df[0].tolist()[:20]
        
        result = {
            'keyword': keyword,
            'region': region,
            'interest_over_time': interest_dict,
            'related_top': related_top,
            'related_rising': related_rising,
            'trending_searches': trending_list,
            'fetched_at': datetime.now().isoformat()
        }
        
        print(json.dumps(result))
        
    except Exception as e:
        error_result = {
            'error': str(e),
            'keyword': keyword,
            'region': region
        }
        print(json.dumps(error_result))
        sys.exit(1)

if __name__ == '__main__':
    if len(sys.argv) < 2:
        print(json.dumps({'error': 'Keyword required'}))
        sys.exit(1)
    
    keyword = sys.argv[1]
    region = sys.argv[2] if len(sys.argv) > 2 else 'IN'
    fetch_trends(keyword, region)
```

### B. Sentiment-Aware Transformation

**The Challenge:**
Generic AI outputs don't resonate with specific platforms or audiences. Instagram needs emojis and casual tone, YouTube needs SEO-focused titles.

**The Solution:**
Platform-specific prompt engineering with system instructions that define the AI's persona and output format.

**Implementation:**

**Gemini Service (server/services/geminiService.js):**
```javascript
const { GoogleGenerativeAI } = require('@google/generative-ai');

class GeminiService {
  constructor() {
    this.genAI = new GoogleGenerativeAI(process.env.GEMINI_API_KEY);
  }
  
  async generateContent(transcript, trends, platform) {
    const model = this.genAI.getGenerativeModel({ model: 'gemini-pro' });
    
    const systemPrompts = {
      instagram: `You are a viral Instagram influencer targeting Indian millennials and Gen Z.
                  Create an engaging caption that:
                  - Uses 3-5 relevant emojis
                  - Includes a hook in the first line
                  - Incorporates these trending keywords naturally: ${trends.related_top.join(', ')}
                  - Adds 10-15 relevant hashtags at the end
                  - Keeps it under 2200 characters
                  - Uses casual, relatable language
                  - Includes a call-to-action`,
      
      youtube: `You are a YouTube SEO expert specializing in Indian content.
                Create an optimized title and description that:
                - Title: 60-70 characters, includes trending keyword: ${trends.related_top[0]}
                - Description: 150-200 words
                - Naturally incorporates these keywords: ${trends.related_top.join(', ')}
                - Includes timestamps if applicable
                - Adds relevant tags (10-15)
                - Uses search-friendly language
                - Includes CTAs for likes, comments, subscribe`,
      
      facebook: `You are a community manager for Indian Facebook groups.
                 Create a post that:
                 - Starts with an engaging question or statement
                 - Incorporates trending topics: ${trends.related_top.join(', ')}
                 - Encourages comments and discussion
                 - Uses conversational tone
                 - Includes 2-3 relevant hashtags
                 - Keeps it under 500 words
                 - Ends with a clear call-to-action`
    };
    
    const prompt = `${systemPrompts[platform]}
    
    Transcript to transform:
    ${transcript}
    
    Additional trending searches in India right now:
    ${trends.trending_searches.slice(0, 5).join(', ')}
    
    Generate the ${platform} content now:`;
    
    const result = await model.generateContent(prompt);
    const response = await result.response;
    return response.text();
  }
  
  async generateMultiPlatform(transcript, trends) {
    const platforms = ['instagram', 'facebook', 'youtube'];
    const results = {};
    
    for (const platform of platforms) {
      results[platform] = await this.generateContent(transcript, trends, platform);
    }
    
    return results;
  }
}

module.exports = new GeminiService();
```

### C. Data-Driven Scheduling

**The Challenge:**
Posting at random times leads to poor engagement. We need to identify when interest in a topic is rising.

**The Solution:**
Analyze interest-over-time data using moving averages and slope detection to find the optimal posting window.

**Implementation:**

**Scheduler Algorithm (python/scheduler_algo.py):**
```python
import numpy as np
from datetime import datetime, timedelta

def calculate_optimal_time(interest_data):
    """
    Analyzes interest-over-time data to find optimal posting time.
    
    Args:
        interest_data: Dict of {timestamp: interest_value}
    
    Returns:
        Dict with recommendation and reasoning
    """
    if not interest_data or len(interest_data) < 3:
        return {
            'recommended': False,
            'reason': 'Insufficient data',
            'post_time': None
        }
    
    # Convert to numpy array
    values = np.array(list(interest_data.values()))
    timestamps = list(interest_data.keys())
    
    # Calculate 3-day moving average
    window = 3
    moving_avg = np.convolve(values, np.ones(window)/window, mode='valid')
    
    # Calculate slope (rate of change)
    slopes = np.diff(moving_avg)
    
    # Find rising trends (positive slope)
    rising_indices = np.where(slopes > 0)[0]
    
    if len(rising_indices) == 0:
        return {
            'recommended': False,
            'reason': 'No rising trend detected',
            'post_time': 'Wait for trend spike',
            'current_interest': int(values[-1])
        }
    
    # Find the steepest rise
    steepest_idx = rising_indices[np.argmax(slopes[rising_indices])]
    
    # Recommend posting 2-4 hours before predicted peak
    recommended_time = datetime.now() + timedelta(hours=3)
    
    return {
        'recommended': True,
        'reason': 'Rising trend detected',
        'post_time': recommended_time.isoformat(),
        'current_interest': int(values[-1]),
        'predicted_peak': int(values[-1] * 1.5),
        'confidence': 'high' if slopes[steepest_idx] > 10 else 'medium'
    }

def get_platform_best_times(platform):
    """Returns platform-specific best posting times for India."""
    best_times = {
        'instagram': ['19:00', '21:00', '12:00'],  # Evening and lunch
        'facebook': ['13:00', '20:00', '09:00'],   # Lunch, evening, morning
        'youtube': ['18:00', '20:00', '14:00']     # Post-work hours
    }
    return best_times.get(platform, ['12:00', '18:00'])
```

**Node.js Integration (server/controllers/scheduleController.js):**
```javascript
const trendService = require('../services/trendService');

async function getScheduleRecommendation(req, res) {
  try {
    const { keyword, platform } = req.body;
    
    // Fetch trends
    const trends = await trendService.fetchTrends(keyword);
    
    // Calculate optimal time
    const timing = trendService.calculateOptimalTime(
      trends.interest_over_time
    );
    
    // Get platform-specific best times
    const platformTimes = {
      instagram: ['19:00', '21:00', '12:00'],
      facebook: ['13:00', '20:00', '09:00'],
      youtube: ['18:00', '20:00', '14:00']
    };
    
    res.json({
      trend_based: timing,
      platform_best_times: platformTimes[platform],
      recommendation: timing.recommended 
        ? `Post in ${timing.postIn} to catch rising trend`
        : `Wait for trend spike, or post at ${platformTimes[platform][0]}`
    });
    
  } catch (error) {
    res.status(500).json({ error: error.message });
  }
}

module.exports = { getScheduleRecommendation };
```

## AI Workflows

### Content Generation Flow
```
1. User uploads transcript + selects topic keyword
2. Node.js validates input and extracts keyword
3. Backend spawns Python child process:
   exec('python3 trend_fetcher.py "keyword" "IN"')
4. Python Pytrends fetches:
   - Interest over time (7 days)
   - Related top queries
   - Related rising queries
   - Current trending searches in India
5. Python outputs JSON to stdout
6. Node.js captures and parses JSON
7. Backend constructs platform-specific prompts:
   - Instagram: Influencer persona + trends
   - YouTube: SEO expert persona + trends
   - Facebook: Community manager persona + trends
8. Send prompts to Google Gemini API
9. Gemini generates 3 platform-specific posts
10. Backend stores in MongoDB
11. Return to frontend for user review
```

### Smart Scheduling Flow
```
1. User requests scheduling recommendation
2. Backend retrieves interest_over_time data
3. Calculate 3-day moving average:
   movingAvg[i] = (values[i-2] + values[i-1] + values[i]) / 3
4. Detect slope (rate of change):
   slope[i] = movingAvg[i] - movingAvg[i-1]
5. Find positive slopes (rising trends)
6. If rising trend detected:
   - Recommend posting in 2-4 hours
   - Confidence: high/medium based on slope steepness
7. If no rising trend:
   - Fall back to platform-specific best times
   - Instagram: 7PM, 9PM, 12PM IST
   - YouTube: 6PM, 8PM, 2PM IST
   - Facebook: 1PM, 8PM, 9AM IST
8. Return recommendation with reasoning
```

### Publishing Flow
```
1. User approves content and schedule
2. Backend stores in scheduled_posts collection
3. Cron job checks for posts due in next 5 minutes
4. For each due post:
   - Retrieve platform credentials
   - Call platform API (Instagram/Facebook/YouTube)
   - Update status to 'published'
   - Store post ID for analytics
5. Background job fetches engagement metrics hourly
6. Update performance_metrics in database
```

## Frontend Architecture

### Key Components

**Dashboard Component:**
```javascript
- Overview stats (posts generated, scheduled, published)
- Quick action buttons (New Content, View Trends)
- Recent content list
- Upcoming scheduled posts
```

**Upload Component:**
```javascript
- Transcript text area
- File upload for .txt, .docx
- Keyword input field
- Platform selection checkboxes
- Generate button
```

**Content Preview Component:**
```javascript
- Platform tabs (Instagram, Facebook, YouTube)
- Editable content fields
- Trending keywords highlight
- Character count display
- Preview mockup for each platform
```

**Schedule Component:**
```javascript
- Calendar view
- Trend-based recommendation badge
- Platform-specific best times
- Manual time picker
- Schedule/Publish buttons
```

**Analytics Component:**
```javascript
- Performance charts (engagement, reach)
- Trend correlation graphs
- Best performing posts
- Keyword effectiveness
```

### UI/UX Flow
```
Login → Dashboard → Upload Transcript → Select Platforms → 
View Trends → Generate Content → Review & Edit → 
Schedule → Publish → Analytics
```

## Security & Privacy

### Authentication
- JWT-based authentication with refresh tokens
- Password hashing using bcrypt
- OAuth2 for social media platform connections
- Session management with Redis

### Data Protection
- Environment variables for API keys
- Encrypted storage for social media credentials
- HTTPS for all API communications
- Rate limiting to prevent abuse
- Input sanitization to prevent injection attacks

### API Key Management
```javascript
// .env file
GEMINI_API_KEY=your_gemini_key
MONGODB_URI=your_mongodb_connection
JWT_SECRET=your_jwt_secret
INSTAGRAM_CLIENT_ID=your_instagram_id
FACEBOOK_APP_ID=your_facebook_id
YOUTUBE_API_KEY=your_youtube_key
```

## Deployment Strategy

### Development Environment
```bash
# Local setup
npm install
pip install pytrends google-generativeai
mongod --dbpath ./data/db
npm run dev
```

### Production Deployment

**Infrastructure:**
- Cloud provider: AWS/GCP/Azure
- Frontend: Vercel/Netlify
- Backend: AWS EC2 / Google Cloud Run
- Database: MongoDB Atlas
- Python: AWS Lambda for trend fetching (optional)

**Scaling Strategy:**
- Load balancer for Node.js instances
- Python worker pool for parallel trend fetching
- MongoDB replica sets for high availability
- Redis caching for frequently accessed trends
- CDN for static assets

**Monitoring:**
- Application logs (Winston/Morgan)
- Error tracking (Sentry)
- Performance monitoring (New Relic)
- Uptime monitoring (Pingdom)

## Future Enhancements
- Video content generation from transcripts
- Voice-to-text transcript creation
- Advanced image generation for posts
- Multi-language UI (Hindi, Tamil, Telugu)
- Mobile applications (React Native)
- Browser extension for quick content generation
- Collaboration features for teams
- White-label solutions for agencies
- Advanced analytics with ML predictions
