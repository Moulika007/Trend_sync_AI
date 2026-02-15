# PulseVibe AI - Requirements

## Project Overview
An AI-powered content lifecycle manager specifically designed for Indian creators. PulseVibe AI transforms raw content into platform-optimized posts using real-time Google Search trends from India, ensuring maximum reach and engagement.

## Core Features

### 1. Trend Ingestion (Real-Time India-Specific Trends)
- Live fetching of Google Search trends via Pytrends library
- India-specific geo-targeting for localized content
- Trending keyword extraction and ranking
- Interest-over-time data collection for timing optimization
- Related queries and rising topics discovery
- Trend snapshot storage for historical analysis

### 2. Multi-Platform Content Creation
- AI-powered transformation of raw transcripts into platform-specific content
- Instagram: Captions with hashtags, carousel text, story scripts
- Facebook: Engaging posts with call-to-actions, group-optimized content
- YouTube: SEO-optimized titles, descriptions, tags, and timestamps
- Platform-specific character limits and formatting
- Automatic trending keyword injection into captions

### 3. Sentiment-Aware Transformation
- Context-aware prompt engineering with Google Gemini API
- Platform-specific persona modeling (Instagram Influencer vs YouTube SEO Expert)
- Tone adaptation based on content type and target audience
- Emotional resonance optimization for Indian audiences
- Cultural context preservation during transformation
- Multi-language support (Hindi, English, Hinglish)

### 4. Search SEO Optimization
- Automatic injection of trending keywords into content
- Keyword density optimization without compromising readability
- Hashtag generation based on current search volume
- Meta description and title tag suggestions
- Search intent matching for better discoverability
- Competitor keyword analysis

### 5. Smart Scheduler (Data-Driven Posting)
- Time-to-post suggestions based on live search volume spikes
- Moving average calculation of search interest trends
- Slope detection for rising interest identification
- Optimal posting window recommendations
- Platform-specific best time analysis
- Automated scheduling with trend-based triggers

## Technical Requirements

### Technology Stack
| Component | Technology | Role |
|-----------|-----------|------|
| **Frontend** | **React.js** | User dashboard for transcript upload and post generation |
| **Backend** | **Node.js & Express** | Main server managing requests and connecting AI to trends |
| **Trend Engine** | **Python (Pytrends)** | Specialized script fetching real-time Google Search trends from India |
| **AI Brain** | **Google Gemini API** | Generates captions, transforms content, provides design advice |
| **Database** | **MongoDB** | Stores creator history, trending keywords, scheduled posts |
| **Inter-process** | **Child Process / ShellJS** | Enables Node.js to communicate with Python script |

### Performance
- Trend data fetching: < 3 seconds
- Content generation: < 5 seconds per platform
- Python-Node.js communication: < 1 second
- Dashboard load time: < 2 seconds
- Support for 500+ concurrent creators
- 99.5% uptime SLA

### Security & Privacy
- API key encryption for Gemini and social platforms
- User content encryption at rest
- Secure credential storage for social media accounts
- Rate limiting to prevent API abuse
- Content version history and backup

### Integration
- Google Trends API (via Pytrends)
- Google Gemini API for content generation
- Instagram Graph API
- Facebook Graph API
- YouTube Data API v3
- MongoDB Atlas for cloud database

### Scalability
- Horizontal scaling for Node.js backend
- Python worker pool for parallel trend fetching
- MongoDB replica sets for data redundancy
- Redis caching for frequently accessed trends
- Queue system for batch content generation

## User Personas

### Indian YouTube Creator
- Needs: SEO-optimized titles/descriptions, trending topic integration, multi-language support
- Pain Points: Missing trending topics, poor search visibility, time-consuming optimization

### Instagram Influencer
- Needs: Engaging captions, relevant hashtags, story scripts, carousel content
- Pain Points: Keeping up with trends, hashtag research, consistent posting schedule

### Facebook Content Manager
- Needs: Community-focused posts, engagement-driven content, group optimization
- Pain Points: Low reach, declining engagement, content fatigue

### Multi-Platform Creator
- Needs: Single transcript to multiple platform outputs, unified scheduling, trend insights
- Pain Points: Repurposing content manually, missing platform-specific nuances, time management

## Technical Implementation Details

### A. Bridging Node.js and Python
The biggest technical hurdle is connecting the Python-based `pytrends` library with our Node.js backend. We rectify this by using the `child_process` module in Node.js to execute the Python script as a background worker. This script returns a **JSON object** of current trends, which our backend then feeds into the Gemini API.

**Implementation:**
```javascript
const { exec } = require('child_process');

function fetchTrends(keyword, region = 'IN') {
  return new Promise((resolve, reject) => {
    exec(`python3 trend_fetcher.py "${keyword}" "${region}"`, (error, stdout, stderr) => {
      if (error) reject(error);
      resolve(JSON.parse(stdout));
    });
  });
}
```

### B. Sentiment-Aware Transformation
To ensure the content isn't just generic, we use **Prompt Engineering** with Gemini. We don't just send the transcript; we send 'System Instructions' that define the character of an Instagram Influencer vs. a YouTube SEO expert. This ensures the output is contextually accurate to the platform.

**Platform-Specific Prompts:**
- Instagram: "Act as a viral Instagram influencer targeting Indian millennials. Create engaging captions with emojis and trending hashtags."
- YouTube: "Act as a YouTube SEO expert. Create titles and descriptions optimized for Indian search queries with trending keywords."
- Facebook: "Act as a community manager. Create posts that encourage discussion and engagement in Indian Facebook groups."

### C. Data-Driven Scheduling
Instead of manual scheduling, we use the `interest_over_time` data from Pytrends. We calculate the moving average of search interest for the user's topic and identify the 'Slope' of rising interest. The system suggests posting at the start of this slope for maximum reach.

**Algorithm:**
1. Fetch 7-day interest_over_time data for topic
2. Calculate 3-day moving average
3. Detect positive slope (rising trend)
4. Recommend posting 2-4 hours before predicted peak
5. Factor in platform-specific best times (evening for Instagram, morning for YouTube)

## Success Metrics
- Content generation speed: 80% faster than manual creation
- Trend accuracy: 90%+ relevant trending keywords
- SEO improvement: 50%+ increase in search visibility
- User satisfaction: 4.5+ star rating
- Time saved: 70% reduction in content optimization time
- Retention rate: 75%+ after 3 months
