# PulseVibe AI

An AI-powered content lifecycle manager specifically designed for Indian creators. Transform raw transcripts into platform-optimized posts using real-time Google Search trends for maximum reach and engagement.

## What You're Building

### 1. **Trend Ingestion Engine**
Real-time India-specific Google Search trends:
- Live trend fetching via Pytrends library
- Geo-targeted to Indian audience
- Trending keyword extraction and ranking
- Interest-over-time data for timing optimization
- Related queries and rising topics discovery
- Historical trend analysis

### 2. **Multi-Platform Content Creator**
Transform one transcript into multiple platform-optimized posts:
- **Instagram**: Engaging captions with hashtags, carousel text, story scripts
- **Facebook**: Community-focused posts with CTAs, group-optimized content
- **YouTube**: SEO-optimized titles, descriptions, tags, and timestamps
- Automatic trending keyword injection
- Platform-specific character limits and formatting
- Cultural context preservation for Indian audiences

### 3. **Sentiment-Aware AI Transformation**
Context-aware content generation with Google Gemini:
- Platform-specific persona modeling (Influencer vs SEO Expert)
- Tone adaptation based on content type and audience
- Emotional resonance optimization for Indian viewers
- Multi-language support (Hindi, English, Hinglish)
- Prompt engineering for contextually accurate outputs
- Cultural nuance preservation

### 4. **Search SEO Optimizer**
Maximize discoverability with intelligent keyword integration:
- Automatic trending keyword injection into captions
- Keyword density optimization
- Hashtag generation based on current search volume
- Meta description and title tag suggestions
- Search intent matching
- Competitor keyword analysis

### 5. **Smart Scheduler**
Data-driven posting recommendations:
- Time-to-post suggestions based on live search volume spikes
- Moving average calculation of search interest
- Slope detection for rising trend identification
- Optimal posting window recommendations
- Platform-specific best time analysis
- Automated scheduling with trend-based triggers

## Key Features

🔥 **Real-Time Trends** - Live Google Search trends from India
🤖 **AI Transformation** - Google Gemini-powered content generation
📱 **Multi-Platform** - Instagram, Facebook, YouTube optimization
🎯 **SEO Injection** - Automatic trending keyword integration
⏰ **Smart Timing** - Data-driven posting recommendations
🇮🇳 **India-Focused** - Localized content for Indian audiences
🌐 **Multi-Language** - Hindi, English, Hinglish support
📊 **Trend Analytics** - Interest-over-time insights

## Technology Stack

| Component | Technology | Role |
|-----------|-----------|------|
| **Frontend** | **React.js** | User dashboard for transcript upload and post generation |
| **Backend** | **Node.js & Express** | Main server managing requests and connecting AI to trends |
| **Trend Engine** | **Python (Pytrends)** | Specialized script fetching real-time Google Search trends from India |
| **AI Brain** | **Google Gemini API** | Generates captions, transforms content, provides design advice |
| **Database** | **MongoDB** | Stores creator history, trending keywords, scheduled posts |
| **Inter-process** | **Child Process / ShellJS** | Enables Node.js to communicate with Python script |

### How They Work Together

**The PulseVibe Flow:**
1. Creator uploads transcript via React dashboard
2. Node.js backend receives request
3. Backend spawns Python child process to fetch India trends via Pytrends
4. Python returns JSON of trending keywords
5. Node.js sends transcript + trends to Google Gemini API
6. Gemini generates platform-specific content with trending keywords
7. MongoDB stores generated content and trend snapshots
8. Smart scheduler analyzes interest-over-time data
9. System recommends optimal posting times
10. Creator reviews and publishes to platforms

## Getting Started

### Prerequisites
- Node.js 18+
- Python 3.10+
- MongoDB 6+
- Google Gemini API key
- Social media platform API credentials

### Quick Start
```bash
# Clone the repository
git clone https://github.com/yourusername/pulsevibe-ai.git
cd pulsevibe-ai

# Install Node.js dependencies
npm install

# Install Python dependencies
pip install pytrends google-generativeai

# Set up environment variables
cp .env.example .env
# Add your GEMINI_API_KEY, MONGODB_URI, and social platform credentials

# Start MongoDB
mongod --dbpath ./data/db

# Start development server
npm run dev
```

## Project Structure
```
pulsevibe-ai/
├── client/                    # React frontend
│   ├── src/
│   │   ├── components/       # UI components
│   │   ├── pages/            # Dashboard, Editor, Analytics
│   │   └── services/         # API calls
├── server/                    # Node.js backend
│   ├── routes/               # API endpoints
│   ├── controllers/          # Business logic
│   ├── models/               # MongoDB schemas
│   └── middleware/           # Auth, validation
├── python/                    # Python trend engine
│   ├── trend_fetcher.py      # Pytrends script
│   └── scheduler_algo.py     # Timing optimization
├── config/                    # Configuration files
└── docs/                      # Documentation
```

## Use Cases

### For Indian YouTube Creators
- SEO-optimized titles and descriptions with trending keywords
- Automatic timestamp generation from transcripts
- Multi-language support for Hindi/English content
- Optimal upload timing based on search trends

### For Instagram Influencers
- Viral caption generation with trending hashtags
- Story script creation from long-form content
- Carousel post text optimization
- Best posting time recommendations

### For Facebook Content Managers
- Community-focused posts that drive engagement
- Group-optimized content with CTAs
- Trend-aware topic suggestions
- Cultural context preservation

### For Multi-Platform Creators
- Single transcript to multiple platform outputs
- Unified trend dashboard
- Cross-platform scheduling
- Performance comparison analytics

## Technical Implementation

### A. Bridging Node.js and Python
The biggest technical hurdle is connecting the Python-based `pytrends` library with our Node.js backend. We rectify this by using the `child_process` module in Node.js to execute the Python script as a background worker. This script returns a **JSON object** of current trends, which our backend then feeds into the Gemini API.

### B. Sentiment-Aware Transformation
To ensure the content isn't just generic, we use **Prompt Engineering** with Gemini. We don't just send the transcript; we send 'System Instructions' that define the character of an Instagram Influencer vs. a YouTube SEO expert. This ensures the output is contextually accurate to the platform.

### C. Data-Driven Scheduling
Instead of manual scheduling, we use the `interest_over_time` data from Pytrends. We calculate the moving average of search interest for the user's topic and identify the 'Slope' of rising interest. The system suggests posting at the start of this slope for maximum reach.

## Roadmap

**Phase 1: Core Engine** (Months 1-2)
- Python-Node.js bridge implementation
- Pytrends integration for India
- Google Gemini API integration
- Basic React dashboard

**Phase 2: Platform Integration** (Months 3-4)
- Instagram, Facebook, YouTube APIs
- Multi-platform content generation
- Smart scheduler algorithm
- MongoDB data persistence

**Phase 3: Intelligence Layer** (Months 5-6)
- Advanced trend analysis
- Performance analytics
- A/B testing features
- Mobile-responsive design

## Contributing
We welcome contributions! Please see CONTRIBUTING.md for guidelines.

## License
MIT License - see LICENSE.md for details

## Support
- Documentation: docs.pulsevibe.ai
- Email: support@pulsevibe.ai
- Discord: discord.gg/pulsevibe


