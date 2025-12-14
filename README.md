# picking-tracker
NJ3 Real Time Picking Tracker
# NJ3 Picking Tracker

A web-based dashboard for tracking warehouse picking batches in real-time, featuring productivity analytics and multi-user access for warehouse operations teams.

## Overview

The NJ3 Picking Tracker is designed to manage B2B and B2C picking operations across multiple client accounts. It provides real-time tracking of batch progress, employee productivity metrics, and completion analytics with automatic time tracking in Eastern Time (America/New_York).

## Features

### 📦 Batch Management
- **Start Batch**: Sign out picking batches with automatic timestamp
- **Complete Batch**: Sign in completed batches with duration tracking
- **Case-Insensitive Batch IDs**: Auto-uppercase conversion for consistent tracking
- **Exception Tracking**: Log issues like short picks, damaged items, or missing inventory

### 📊 Real-Time Dashboards
1. **Start/Complete Batch Tab**: Dual-panel interface for starting new batches and completing in-progress work
2. **In Progress Tab**: Live view of all active picking batches
3. **Productivity Tab**: Daily performance rankings by employee with lines-per-hour metrics
4. **Completed Batches Archive**: Historical record of all completed work

### 📈 Analytics & Reporting
- **Lines Per Hour Calculation**: Automatic productivity metrics per employee
- **Top Performer Rankings**: Visual charts with gold/silver/bronze rankings
- **Daily Performance Tracking**: Filters to show today's productivity only
- **Excel Export**: Download completed batches and productivity reports as CSV

### 🕐 Time Management
- All timestamps recorded in **Eastern Time (America/New_York)**
- Automatic duration calculation in minutes
- Human-readable date/time display (MM/DD/YYYY h:mm AM/PM)

## Supported Accounts

The system tracks picking operations for 11 client accounts:
- Pierre Fabre
- BR (Biologique Recherche)
- Promix
- Maelys
- Natura
- Granado
- VBB (VB Beauty)
- Maison (Lampe Berger)
- Yves Rocher
- LMC (LimeLife)

## System Architecture

### Frontend (HTML/JavaScript)
- **Single-page application** with tab-based navigation
- **Real-time updates** via fetch API calls to Google Apps Script
- **Automatic refresh** capabilities on all data tables
- **Client-side validation** and error handling
- **Responsive design** optimized for desktop and tablet use

### Backend (Google Apps Script)
- **Google Sheets** as database with two main sheets:
  - `InProgress`: Active batches currently being picked
  - `Completed`: Historical archive of finished batches
- **RESTful API** endpoints via doGet() and doPost()
- **Data normalization**:
  - Employee names standardized to Title Case
  - Batch IDs compared case-insensitively
  - Automatic removal of employee ID codes (e.g., NJ03XXXXX)

## Setup Instructions

### 1. Google Sheets Setup
1. Create a new Google Sheets spreadsheet
2. The script will automatically create two sheets:
   - **InProgress**: Columns - Batch ID, Account, Employee ID, Quantity, SKU Count, Start Time, Status
   - **Completed**: Columns - Batch ID, Account, Employee ID, Quantity, SKU Count, Start Time, End Time, Duration (min), Exception, Completion Date, Status

### 2. Deploy Google Apps Script
1. Open your Google Sheet
2. Go to **Extensions > Apps Script**
3. Delete any existing code
4. Paste the entire Google Apps Script code
5. Click **Deploy > New Deployment**
6. Select type: **Web app**
7. Settings:
   - Execute as: **Me**
   - Who has access: **Anyone** (or your organization)
8. Click **Deploy** and authorize the script
9. Copy the **Web app URL**

### 3. Configure HTML Dashboard
1. Open the HTML file in a text editor
2. Find line with `const SCRIPT_URL = '...'`
3. Replace the URL with your Web app URL from step 2.9
4. Save the file

### 4. Host the HTML File
**Option A: GitHub Pages (Recommended)**
1. Create a new GitHub repository
2. Upload the HTML file (rename to `index.html`)
3. Go to Settings > Pages
4. Select branch: `main`, folder: `/root`
5. Save and wait for deployment
6. Access via: `https://yourusername.github.io/repositoryname`

**Option B: Google Drive**
1. Upload HTML file to Google Drive
2. Right-click > Share > Change to "Anyone with the link"
3. Use Google Drive File Stream or publish as web page

**Option C: Local Server**
- Simply open the HTML file in a web browser
- Note: CORS may require the Google Apps Script to allow your local domain

## How to Use

### Starting a Batch
1. Navigate to **Start/Complete Batch** tab
2. In the "START NEW BATCH" section:
   - Scan or enter **Batch/Pull ID** (case insensitive)
   - Select **Account** from dropdown
   - Scan or enter **Employee ID/Name**
   - Enter **Batch Quantity** (total units)
   - Enter **SKU Count** (number of unique line items)
3. Click **Start Batch**
4. Batch appears in "In Progress" list immediately

### Completing a Batch
1. In the "COMPLETE BATCH" section:
   - Scan the **Batch/Pull ID** (case insensitive - will auto-uppercase)
   - Select **Exception** if applicable (optional)
2. Click **Complete Batch** or press **Enter**
3. System automatically:
   - Calculates duration in minutes
   - Moves batch to Completed archive
   - Removes from In Progress list
   - Updates productivity metrics

### Viewing Productivity
1. Navigate to **Productivity** tab
2. View real-time chart showing:
   - Employee rankings (top performers highlighted)
   - Lines per hour for each employee
   - Number of batches completed
3. Click **Download to Excel** for detailed CSV report including:
   - Date, Employee, Total Batches, Total Quantities
   - Total Lines, Total Minutes, Averages
   - Lines/Hour and Qty/Hour metrics

### Accessing Completed Batches
1. Navigate to **Completed Batches Archive** tab
2. View all historical batches (newest first)
3. Click **Download to Excel** for CSV export
4. Click **🔄 Refresh** to update the list

## Data Normalization Features

### Employee Name Normalization
- Converts to Title Case (e.g., "john smith" → "John Smith")
- Removes employee ID codes (e.g., "NJ03XXXXX")
- Trims extra whitespace
- Ensures consistent reporting across productivity metrics

### Batch ID Normalization
- **Case-insensitive matching**: PK-157025 = pk-157025 = Pk-157025
- **Auto-uppercase input fields**: Batch IDs automatically convert to uppercase as you type
- **Trimmed whitespace**: Prevents duplicate entry errors

### Date/Time Standardization
- All dates stored in Eastern Time (America/New_York)
- Consistent formatting: MM/DD/YYYY h:mm AM/PM
- Completion dates stored as M/D/YYYY for daily filtering

## API Endpoints

### GET Requests
```
?action=getInProgress
```
Returns all batches currently in progress

```
?action=getCompleted
```
Returns all completed batches (newest first)

```
?action=getProductivityChart
```
Returns today's productivity metrics by employee

### POST Requests
**Start Batch:**
```json
{
  "action": "startBatch",
  "batchId": "PK-157025",
  "account": "Pierre Fabre",
  "employeeId": "John Smith",
  "qty": 500,
  "skuCount": 25
}
```

**Complete Batch:**
```json
{
  "action": "completeBatch",
  "batchId": "PK-157025",
  "exception": "Short Picked"
}
```

## Exception Types

The system tracks the following exception types:
- **Short Picked**: Insufficient inventory to fulfill order
- **Damaged Item**: Product damaged and cannot be shipped
- **Item Not Found**: SKU not located in warehouse
- **Wrong Location**: Item found in incorrect bin location

## Productivity Calculations

### Lines Per Hour
```
Lines Per Hour = (Total SKU Count / Total Duration in Minutes) × 60
```

### Duration Calculation
```
Duration (minutes) = End Time - Start Time
```
*Rounded to nearest minute*

## Troubleshooting

### Common Issues

**Problem**: Batch ID not found when completing
- **Solution**: Check spelling and case (system auto-converts to uppercase)
- Verify batch was started in the same system
- Check "In Progress" tab to confirm batch exists

**Problem**: Data not loading in dashboard
- **Solution**: 
  - Check SCRIPT_URL is correct in HTML file
  - Verify Google Apps Script is deployed as web app
  - Check browser console for CORS errors
  - Click 🔄 Refresh button

**Problem**: Times showing incorrectly
- **Solution**: All times are in Eastern Time (America/New_York)
- Check your Google Sheet timezone settings
- Verify date/time formatting in Sheets

**Problem**: Employee names appearing inconsistently
- **Solution**: System auto-normalizes to Title Case
- Remove any employee ID codes from name field
- Use consistent naming (first and last name)

## Best Practices

### For Warehouse Staff
1. **Scan batch IDs** when possible to avoid typos
2. **Start batches immediately** when beginning pick
3. **Complete batches promptly** after finishing
4. **Log exceptions** accurately for quality tracking
5. **Use consistent names** for accurate productivity tracking

### For Supervisors
1. **Monitor In Progress tab** for batch status
2. **Review Productivity tab daily** for performance insights
3. **Export data regularly** for backup and analysis
4. **Investigate exceptions** for process improvement
5. **Check for stuck batches** (very long durations)

## Data Privacy & Security

- All data stored in Google Sheets owned by your organization
- Access controlled via Google Apps Script deployment settings
- No external databases or third-party services
- Employee data normalized to remove sensitive ID codes
- HTTPS encryption via Google Apps Script web app

## Maintenance

### Regular Tasks
- **Weekly**: Export completed batches for backup
- **Monthly**: Archive old completed batches to separate sheet
- **Quarterly**: Review exception trends and process improvements

### Updating the System
1. **Google Apps Script**: Edit script, save, and deploy new version
2. **HTML Dashboard**: Update file and re-upload to hosting service
3. **Account List**: Update dropdown options in HTML if clients change

## Technical Requirements

- **Modern web browser** (Chrome, Firefox, Safari, Edge)
- **Google Account** with access to Google Sheets
- **Internet connection** for real-time updates
- **Screen resolution**: 1280x720 or higher recommended

## Future Enhancement Ideas

- [ ] Mobile app version for floor operations
- [ ] Barcode scanner integration
- [ ] SMS/email alerts for stuck batches
- [ ] Multi-warehouse support
- [ ] Advanced analytics dashboard
- [ ] Integration with WMS systems
- [ ] Photo upload for exceptions
- [ ] Automated daily reports

## Support & Feedback

For issues, questions, or feature requests:
1. Check the Troubleshooting section above
2. Review browser console for error messages
3. Contact your warehouse IT administrator
4. Document issue with screenshots and batch IDs

## Version History

**Version 1.0** (Current)
- Initial release with batch tracking
- Productivity analytics
- Case-insensitive batch ID matching
- Employee name normalization
- Excel export functionality
- Eastern Time timezone support

---

**Created by**: Roberto (Warehouse Operations Freelancer)
**Last Updated**: December 2024  
**Technology Stack**: Google Apps Script, Google Sheets, HTML5, JavaScript, CSS3
