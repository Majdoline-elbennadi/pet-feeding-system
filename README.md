# Pet Feeder Control System 🐾

A professional IoT pet feeder control web application built with vanilla JavaScript, featuring real-time scheduling, meal management, and seamless Supabase integration. This system allows pet owners to control their automated pet feeder remotely with an intuitive, mobile-responsive interface.

## 📋 Table of Contents

- [Overview](#overview)
- [Features](#features)
- [Architecture](#architecture)
- [Technology Stack](#technology-stack)
- [Project Structure](#project-structure)
- [Installation](#installation)
- [Configuration](#configuration)
- [Usage](#usage)
- [Database Schema](#database-schema)
- [API Integration](#api-integration)
- [Mobile Responsiveness](#mobile-responsiveness)
- [Development](#development)
- [Troubleshooting](#troubleshooting)

## 🎯 Overview

The Pet Feeder Control System is a web-based application that provides a user-friendly interface for managing automated pet feeding schedules. The system supports immediate feeding requests and scheduled feeding times, with full CRUD operations for meal management.

### Key Capabilities

- **Immediate Feeding**: Trigger feeding commands instantly
- **Scheduled Feeding**: Set future feeding times with automatic execution
- **Meal Management**: View, select, and cancel scheduled meals
- **Real-time Updates**: Auto-refreshing status and meal list
- **Pet-specific Limits**: Different maximum quantities for cats (75g) and dogs (200g)
- **Mobile-first Design**: Fully responsive across all device sizes

## ✨ Features

### Core Functionality

1. **Pet Selection**
   - Support for Cats and Dogs
   - Visual pet icons
   - Automatic quantity limit adjustment per pet type

2. **Feeding Modes**
   - **Feed Now**: Immediate feeding command (no time required)
   - **Scheduled**: Future feeding with specific time selection

3. **Meal Scheduling**
   - Create multiple scheduled meals
   - Visual meal list with date/time and quantity
   - Click meals to view detailed status
   - Cancel scheduled meals with confirmation

4. **Status Monitoring**
   - Real-time status display (Pending/Feeded)
   - Detailed meal information (action, amount, scheduled time, creation date)
   - Interactive meal selection for status viewing
   - Auto-refresh every 5 seconds

5. **Input Validation**
   - Minimum quantity validation (1g)
   - Pet-specific maximum limits (Cat: 75g, Dog: 200g)
   - Past time prevention
   - Real-time time input restrictions

### User Experience

- **Loading Indicators**: Visual feedback during API operations
- **Success/Error Messages**: Toast notifications for user actions
- **Interactive Elements**: Hover effects, selected states, smooth transitions
- **Keyboard Support**: Enter key triggers feeding
- **Accessibility**: Semantic HTML, proper labels, focus states

## 🏗️ Architecture

### System Architecture

```
┌─────────────────┐
│   Web Browser   │
│   (Frontend)    │
└────────┬────────┘
         │
         │ HTTP/REST
         │
┌────────▼────────┐
│  Supabase API   │
│   (Backend)     │
└────────┬────────┘
         │
         │ SQL
         │
┌────────▼────────┐
│  PostgreSQL DB  │
│ FeedingRequests │
└─────────────────┘
         │
         │ Poll
         │
┌────────▼────────┐
│   ESP32 Device  │
│  (IoT Hardware) │
└─────────────────┘
```

### Data Flow

1. **User Action** → Frontend captures user input
2. **Validation** → Client-side validation of inputs
3. **API Request** → Supabase REST API call
4. **Database** → PostgreSQL stores/retrieves data
5. **ESP32 Polling** → Hardware device queries pending requests
6. **Execution** → Device executes feeding and updates status
7. **UI Update** → Frontend auto-refreshes to show changes

## 💻 Technology Stack

### Frontend
- **HTML5**: Semantic markup, accessibility features
- **CSS3**: Modern styling with Flexbox, CSS Grid, responsive design
- **Vanilla JavaScript (ES6+)**: No frameworks, pure JavaScript
- **Supabase JS v2**: Client library for database operations

### Backend
- **Supabase**: PostgreSQL-based Backend-as-a-Service
- **PostgreSQL**: Relational database for feeding requests

### External Dependencies
- **Supabase JS Client**: CDN-loaded (`@supabase/supabase-js@2`)

## 📁 Project Structure

```
pet-feeding-system/
├── index.html              # Main HTML file
├── README.md               # Project documentation
├── css/
│   └── styles.css          # All application styles
├── js/
│   └── app.js              # Application logic and Supabase integration
├── assets/
│   ├── cat.avif           # Cat pet image
│   ├── dog.webp           # Dog pet image
│   └── icon.webp          # Favicon
└── html/                   # (Optional) Alternative location
    └── index.html
```

### File Responsibilities

- **`index.html`**: Structure, semantic markup, external resource linking
- **`css/styles.css`**: All visual styling, responsive breakpoints, animations
- **`js/app.js`**: Business logic, API calls, event handlers, state management
- **`assets/`**: Images, icons, static resources

## 🚀 Installation

### Prerequisites

- Modern web browser (Chrome, Firefox, Safari, Edge)
- Web server (for local development)
- Supabase account and project

### Setup Steps

1. **Clone or Download Project**
   ```bash
   git clone <repository-url>
   cd pet-feeding-system
   ```

2. **Serve the Application**
   
   **Option A: Using Python (recommended for quick testing)**
   ```bash
   # Python 3
   python -m http.server 8000
   
   # Python 2
   python -m SimpleHTTPServer 8000
   ```
   
   **Option B: Using Node.js**
   ```bash
   npx http-server -p 8000
   ```
   
   **Option C: Using VS Code Live Server**
   - Install "Live Server" extension
   - Right-click `index.html` → "Open with Live Server"

3. **Access Application**
   - Open browser: `http://localhost:8000`

## ⚙️ Configuration

### Supabase Setup

1. **Create Supabase Project**
   - Go to [supabase.com](https://supabase.com)
   - Create new project
   - Note your project URL and anon key

2. **Database Table Creation**

   Create the `FeedingRequests` table with the following SQL:

   ```sql
   CREATE TABLE "FeedingRequests" (
     id BIGSERIAL PRIMARY KEY,
     action TEXT NOT NULL CHECK (action IN ('now', 'scheduled')),
     schedule_time TIMESTAMP WITH TIME ZONE,
     amount INTEGER NOT NULL CHECK (amount > 0),
     status TEXT NOT NULL DEFAULT 'pending' CHECK (status IN ('pending', 'feeded')),
     created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
   );
   ```

3. **Row Level Security (RLS) Policies**

   Enable RLS and create policies for anonymous access:

   ```sql
   -- Enable RLS
   ALTER TABLE "FeedingRequests" ENABLE ROW LEVEL SECURITY;

   -- Policy for SELECT (read)
   CREATE POLICY "Allow anonymous select on FeedingRequests"
   ON public."FeedingRequests"
   FOR SELECT
   TO anon
   USING (true);

   -- Policy for INSERT (create)
   CREATE POLICY "Allow anonymous insert on FeedingRequests"
   ON public."FeedingRequests"
   FOR INSERT
   TO anon
   WITH CHECK (true);

   -- Policy for DELETE (cancel)
   CREATE POLICY "Allow anonymous delete on FeedingRequests"
   ON public."FeedingRequests"
   FOR DELETE
   TO anon
   USING (true);
   ```

4. **Update Configuration in `js/app.js`**

   Replace with your Supabase credentials:

   ```javascript
   const SUPABASE_URL = "https://your-project.supabase.co";
   const SUPABASE_ANON_KEY = "your-anon-key-here";
   ```

## 📖 Usage

### Basic Workflow

1. **Select Pet Type**
   - Choose Cat or Dog from dropdown
   - Pet image updates automatically
   - Quantity limit adjusts (Cat: 75g, Dog: 200g)

2. **Set Feeding Parameters**
   - **Time** (optional): Leave empty for immediate feeding, or select future time for scheduling
   - **Amount**: Enter quantity in grams (1-75g for cats, 1-200g for dogs)

3. **Execute Feeding**
   - Click "FEED NOW" button
   - System validates inputs
   - Request sent to database
   - Status updates automatically

4. **Manage Scheduled Meals**
   - View all scheduled meals in the list
   - Click any meal to see its status
   - Click "Cancel" to delete a scheduled meal

### Feeding Modes

#### Immediate Feeding
- Leave time field empty
- Enter amount
- Click "FEED NOW"
- Creates `action: "now"` request

#### Scheduled Feeding
- Select future time
- Enter amount
- Click "FEED NOW"
- Creates `action: "scheduled"` request with `schedule_time`

### Status Display

The status panel shows:
- **Status**: Current state (Pending/Feeded)
- **Action**: Feeding type (now/scheduled)
- **Amount**: Food quantity in grams
- **Scheduled Time**: When applicable
- **Created**: Timestamp of request creation

Click any scheduled meal to view its specific status details.

## 🗄️ Database Schema

### Table: `FeedingRequests`

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| `id` | BIGSERIAL | PRIMARY KEY | Auto-incrementing unique identifier |
| `action` | TEXT | NOT NULL, CHECK | Feeding type: 'now' or 'scheduled' |
| `schedule_time` | TIMESTAMP | NULLABLE | ISO timestamp for scheduled feeds |
| `amount` | INTEGER | NOT NULL, CHECK > 0 | Food quantity in grams |
| `status` | TEXT | NOT NULL, DEFAULT 'pending', CHECK | Request status: 'pending' or 'feeded' |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Record creation timestamp |

### Data Flow Example

```javascript
// Immediate Feed
{
  action: "now",
  schedule_time: null,
  amount: 50,
  status: "pending"
}

// Scheduled Feed
{
  action: "scheduled",
  schedule_time: "2024-12-07T15:30:00Z",
  amount: 75,
  status: "pending"
}
```

## 🔌 API Integration

### Supabase Client Methods

#### Create Feeding Request
```javascript
await supabase
  .from('FeedingRequests')
  .insert([requestData])
  .select();
```

#### Read Scheduled Meals
```javascript
await supabase
  .from('FeedingRequests')
  .select('*')
  .eq('status', 'pending')
  .eq('action', 'scheduled')
  .not('schedule_time', 'is', null)
  .order('schedule_time', { ascending: true });
```

#### Delete Meal
```javascript
await supabase
  .from('FeedingRequests')
  .delete()
  .eq('id', mealId)
  .select();
```

#### Get Latest Status
```javascript
await supabase
  .from('FeedingRequests')
  .select('*')
  .order('id', { ascending: false })
  .limit(1);
```

### ESP32 Integration

The ESP32 device should:

1. **Poll for Pending Requests**
   ```sql
   SELECT * FROM "FeedingRequests" 
   WHERE status = 'pending' 
   ORDER BY id ASC 
   LIMIT 1;
   ```

2. **Execute Feeding**
   - For `action = 'now'`: Execute immediately
   - For `action = 'scheduled'`: Execute when `schedule_time <= NOW()`

3. **Update Status**
   ```sql
   UPDATE "FeedingRequests" 
   SET status = 'feeded' 
   WHERE id = <meal_id>;
   ```

## 📱 Mobile Responsiveness

### Breakpoints

- **Small Phones** (≤360px): Optimized spacing, stacked layouts
- **Standard Phones** (≤480px): Full-width elements, vertical stacking
- **Tablets** (481px-768px): Adjusted padding and gaps
- **Desktop** (≥769px): Maximum width container, optimal spacing

### Responsive Features

- **Fluid Typography**: Uses `clamp()` for scalable font sizes
- **Flexible Layouts**: Flexbox with wrap for adaptive arrangements
- **Touch-friendly**: Large tap targets, proper spacing
- **iOS Optimization**: 16px font-size prevents zoom on input focus
- **No Horizontal Scroll**: Prevents overflow on all screen sizes

### Testing Devices

Tested on:
- iPhone SE (375px)
- iPhone 12/13/14 (390px)
- Samsung Galaxy S series (360px)
- iPad Mini (768px)
- iPad Air (820px)
- Desktop (1920px+)

## 🛠️ Development

### Code Organization

The application follows separation of concerns:

- **HTML**: Structure and semantic markup only
- **CSS**: All styling in dedicated stylesheet
- **JavaScript**: Modular functions with clear responsibilities

### JavaScript Functions

| Function | Purpose |
|----------|---------|
| `handleFeedNow()` | Process feed request creation |
| `loadScheduledMeals()` | Fetch and display scheduled meals |
| `loadStatus()` | Load and display meal status |
| `cancelMeal()` | Delete scheduled meal |
| `validateInputs()` | Client-side input validation |
| `displayMealStatus()` | Format and show meal status |
| `selectMeal()` | Handle meal selection UI |
| `showMessage()` | Display toast notifications |
| `updateTimeInputMin()` | Prevent past time selection |

### Adding New Features

1. **New Pet Type**
   - Add option to `<select>` in HTML
   - Update `petImages` and `petMaxQuantities` in `js/app.js`
   - Add corresponding image file to `assets/`

2. **New Validation Rule**
   - Modify `validateInputs()` function
   - Add user feedback as needed

3. **New UI Component**
   - Add HTML structure
   - Style in `css/styles.css`
   - Add event handlers in `js/app.js`

## 🐛 Troubleshooting

### Common Issues

#### Issue: "Error loading status" / "Error loading scheduled meals"

**Solution**: 
- Check Supabase URL and API key in `js/app.js`
- Verify RLS policies are set correctly
- Check browser console for detailed error messages
- Ensure Supabase project is active

#### Issue: Delete operation fails silently

**Solution**:
- Verify DELETE policy exists in Supabase RLS
- Check browser console for error messages
- Ensure meal ID is valid

#### Issue: Images not displaying

**Solution**:
- Verify images are in `assets/` folder
- Check file paths in `js/app.js` (`petImages` object)
- Ensure web server is running (not just opening HTML file)

#### Issue: Time input allows past times

**Solution**:
- Check `updateTimeInputMin()` is called on page load
- Verify setInterval is running (every 60 seconds)
- Clear browser cache and reload

#### Issue: Mobile layout broken

**Solution**:
- Check viewport meta tag in HTML
- Verify CSS media queries are correct
- Test in browser developer tools responsive mode
- Ensure no fixed widths are breaking layout

### Debug Mode

Enable detailed logging by opening browser console (F12):

```javascript
// All errors are logged to console
console.error('Error message', error);
```

### Supabase Debugging

1. **Check Database Logs**: Supabase Dashboard → Logs
2. **Verify Table Structure**: Supabase Dashboard → Table Editor
3. **Test Queries**: Supabase Dashboard → SQL Editor
4. **Check API Keys**: Supabase Dashboard → Settings → API

## 📝 License

This project is part of an IoT pet feeding system implementation. Customize and use as needed for your project.

## 👥 Contributors

Developed as part of SCAP (System Architecture and Programming) course project.

## 🔄 Version History

- **v1.0.0** (Current)
  - Initial release
  - Full CRUD operations
  - Mobile-responsive design
  - Real-time status updates
  - Meal scheduling and cancellation

## 📞 Support

For issues or questions:
1. Check [Troubleshooting](#troubleshooting) section
2. Review browser console errors
3. Verify Supabase configuration
4. Check database RLS policies

---

**Made with ❤️ for pet owners who want the best for their furry friends**

