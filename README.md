# Road Surfacing Progress Tracker with Auto-Cleanup Database

A real-time job tracking system with automatic data cleanup after 24 hours from estimated completion time.

## 🚀 Features

- **Real-time Multi-User Sync**: Multiple users can view and edit jobs simultaneously
- **Auto-Cleanup Database**: Jobs automatically deleted 24 hours after estimated end time
- **QR Code Sharing**: Generate unique links for stakeholders to view progress
- **Delay Tracking**: Log and manage project delays with full audit trail
- **Overdue Detection**: Visual red indicators for segments behind schedule
- **Persistent Storage**: JSON file-based database with automatic backups

## 📋 Setup Instructions

### 1. Start the Backend Server

```bash
node server.js
```

The server will start on `http://localhost:3000`

### 2. Open the Frontend

Open `road-surfacing-tracker-realtime.html` in your web browser.

**Important**: You must open the HTML file via a local web server or by double-clicking (not via `file://` protocol for cross-origin reasons).

For best results, serve it using a simple HTTP server:

```bash
# Option 1: Using Python
python3 -m http.server 8080

# Option 2: Using Node.js http-server (if installed)
npx http-server -p 8080

# Then open: http://localhost:8080/road-surfacing-tracker-realtime.html
```

## 🗄️ Database System

### Storage Location
- All jobs stored in: `jobs.json` (created automatically)
- File-based JSON database for simplicity and portability

### Auto-Cleanup System

The database automatically removes jobs based on the following logic:

1. **Calculation of End Time**:
   - Base duration = Sum of all segment durations
   - Total delays = Sum of all delay durations
   - Estimated end time = Start time + Base duration + Total delays

2. **Deletion Trigger**:
   - Jobs are marked for deletion 24 hours after estimated end time
   - Cleanup runs automatically every hour
   - Manual cleanup available via API: `POST /api/cleanup`

3. **Example Timeline**:
   ```
   Job Start: 9:00 AM
   Total Duration: 3 hours (180 mins)
   Delays: 30 mins
   Estimated End: 12:30 PM
   Deletion Time: 12:30 PM next day (24 hours later)
   ```

## 🔌 API Endpoints

### Jobs Management
- `GET /api/jobs` - Get all jobs
- `GET /api/jobs/:id` - Get specific job
- `POST /api/jobs` - Create new job
- `PUT /api/jobs/:id` - Update job
- `DELETE /api/jobs/:id` - Delete job

### Maintenance
- `POST /api/cleanup` - Manually trigger cleanup

### Example API Usage

```bash
# Get all jobs
curl http://localhost:3000/api/jobs

# Get specific job
curl http://localhost:3000/api/jobs/job_12345

# Create job
curl -X POST http://localhost:3000/api/jobs \
  -H "Content-Type: application/json" \
  -d '{"id":"job_123","name":"Test Job","segments":[]}'

# Trigger manual cleanup
curl -X POST http://localhost:3000/api/cleanup
```

## 📊 Default Job Segments

New jobs come with these pre-configured segments:

1. **Road Closure and Pedestrian Segregation** (30 mins)
2. **Road Planing** (60 mins)
3. **Road Preparation** (45 mins)
4. **Road Surfacing** (60 mins)
5. **Road Lining** (30 mins)

## 🎯 Usage Workflow

### For Site Managers (Edit Mode)

1. **Create Job**: Click "+ New Job" and enter job name
2. **Set Start Time**: Select when work begins
3. **Add/Edit Segments**: Customize phases as needed
4. **Track Progress**: Mark segments complete as work progresses
5. **Log Delays**: Click "⏱️ Add Delay" to record issues
6. **Share Progress**: Click "📱 Share Job" to generate QR code

### For Stakeholders (View Mode)

1. **Scan QR Code** or open shared link
2. **View Live Updates**: Progress updates every 2 seconds
3. **Monitor Timeline**: See estimated completion time
4. **Track Delays**: View all logged delays and reasons

## ⚙️ Configuration

### Cleanup Schedule
Edit `server.js` to change cleanup frequency:

```javascript
// Current: Every hour
setInterval(() => {
    console.log('Running scheduled cleanup...');
    this.cleanup();
}, 60 * 60 * 1000); // Change this value

// Examples:
// Every 30 mins: 30 * 60 * 1000
// Every 6 hours: 6 * 60 * 60 * 1000
// Once per day: 24 * 60 * 60 * 1000
```

### Retention Period
Edit `server.js` to change how long after completion jobs are kept:

```javascript
// Current: 24 hours after end time
const deletionTime = new Date(endTime.getTime() + (24 * 60 * 60 * 1000));

// Examples:
// 12 hours: 12 * 60 * 60 * 1000
// 48 hours: 48 * 60 * 60 * 1000
// 7 days: 7 * 24 * 60 * 60 * 1000
```

## 🔒 Data Persistence

### Backup Strategy
The `jobs.json` file persists across server restarts. To backup:

```bash
# Manual backup
cp jobs.json jobs_backup_$(date +%Y%m%d).json

# Automated daily backup (add to cron)
0 0 * * * cp /path/to/jobs.json /path/to/backups/jobs_$(date +\%Y\%m\%d).json
```

### Data Recovery
If you need to restore data:

```bash
# Stop the server
# Replace jobs.json with backup
cp jobs_backup_20260207.json jobs.json
# Restart the server
```

## 🐛 Troubleshooting

### "Unable to connect to server" error
- Ensure server is running: `node server.js`
- Check server is on port 3000
- Verify no firewall blocking localhost:3000

### Jobs not syncing
- Check browser console for errors
- Verify API URL is correct (http://localhost:3000/api)
- Ensure server is running and accessible

### Jobs not being deleted
- Check server logs for cleanup messages
- Manually trigger: `curl -X POST http://localhost:3000/api/cleanup`
- Verify job has passed 24-hour mark after end time

## 📝 Production Deployment

For production use, consider:

1. **Database Upgrade**: Replace JSON file with PostgreSQL, MongoDB, or SQLite
2. **Authentication**: Add user authentication and job ownership
3. **HTTPS**: Use SSL certificates for secure connections
4. **Process Manager**: Use PM2 or similar to keep server running
5. **Backup System**: Automated database backups
6. **Monitoring**: Add logging and error tracking (e.g., Sentry)

### Quick Production Setup with PM2

```bash
# Install PM2
npm install -g pm2

# Start server with PM2
pm2 start server.js --name road-tracker

# Setup auto-restart on system reboot
pm2 startup
pm2 save
```

## 📄 License

This project is provided as-is for road surfacing job tracking purposes.

## 🤝 Support

For issues or questions, check the server logs and browser console for detailed error messages.
