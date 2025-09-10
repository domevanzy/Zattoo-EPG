# Zattoo EPG Grabber

A Python program that downloads EPG data from Zattoo and saves it as an XMLTV file. This is a Python implementation of the original easyEPG Zattoo grabber.

## Features

- **Login Authentication**: Uses the same API interface as the original program
- **Multiple Countries**: Supports Germany (DE) and Switzerland (CH)
- **Customizable Time Periods**: 1-14 days of EPG data
- **XMLTV Format**: Compatible output format for EPG viewers
- **Detailed Program Information**: Optionally retrieve detailed program information
- **Console Interface**: Simple command-line operation

## Requirements

- Python 3.7 or higher
- Zattoo account (Germany or Switzerland)
- Internet connection

## Installation

1. Clone repository or download files:
```bash
git clone <repository-url>
cd ZattooEPG
```

2. Create virtual environment (recommended):
```bash
python -m venv venv
source venv/bin/activate  # Linux/macOS
# or
venv\Scripts\activate.bat  # Windows
```

3. Install dependencies:
```bash
pip install -r requirements.txt
```

## Configuration

### Configuration File (Recommended)

1. Create a `config.json` file based on the example:
```bash
cp config.example.json config.json
```

2. Edit the `config.json` and enter your Zattoo credentials:
```json
{
    "email": "your-email@example.com",
    "password": "your-password"
}
```

## Usage

### Basic Usage (with configuration file)

```bash
python zattoo_epg.py
```

The program loads credentials from the `config.json` file and downloads 7 days of EPG data for Germany.

# Use interactive login instead of configuration file
```python zattoo_epg.py --interactive
```

# Send EPG data directly to TVHeadend
```python zattoo_epg.py --tvheadend-only
```

# Custom TVHeadend socket path
```python zattoo_epg.py --tvheadend --tvheadend-socket /path/to/tvheadend/xmltv.sock
```

### Advanced Options

```bash
# Swiss EPG for 3 days
python zattoo_epg.py --country CH --days 3

# Without detailed information (much faster - recommended)
python zattoo_epg.py --no-details

# Custom output file
python zattoo_epg.py --output my_epg.xml

# Maximum 14 days
python zattoo_epg.py --days 14

# Custom configuration file
python zattoo_epg.py --config my_config.json

# Interactive login (without configuration file)
python zattoo_epg.py --interactive
```

### Complete Options

```bash
python zattoo_epg.py --help
```

Available parameters:
- `--country/-c`: Country (DE or CH, default: DE)
- `--days/-d`: Number of days (1-14, default: 7)
- `--output/-o`: Output file (default: zattoo_epg.xml)
- `--no-details`: Skip detailed information (faster)
- `--config`: Configuration file with email and password (default: config.json)
- `--interactive`: Use interactive login instead of configuration file
- `--tvheadend`: Send EPG data to TVHeadend after generation
- `--tvheadend-socket`: Path to TVHeadend XMLTV socket (default: /var/lib/tvheadend/epggrab/xmltv.sock)
- `--tvheadend-only`: Send EPG data directly to TVHeadend without saving to file

## Output Format

The program creates an XMLTV file with the following information:

### Channels
- Channel ID
- Channel name
- Channel logo (if available)

### Programs
- **Basic Information**: Title, start/end time, description
- **Additional Information**: Subtitle, production year, country
- **Categories**: Genres and program categories
- **Credits**: Directors and actors
- **Episode Information**: Season and episode numbers
- **Age Rating**: FSK rating
- **Images**: Program posters/images

## Example Output

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE tv SYSTEM "xmltv.dtd">
<tv source-info-url="https://zattoo.com/" source-data-url="https://zattoo.com/" generator-info-name="Zattoo EPG Grabber Python">
  <channel id="cid://tx.zattoo.com/119">
    <display-name lang="de">Das Erste HD</display-name>
    <icon src="https://images.zattic.com/cms/..." />
  </channel>
  
  <programme start="20250909180000 +0000" stop="20250909184500 +0000" channel="cid://tx.zattoo.com/119">
    <title lang="de">Tagesschau</title>
    <desc lang="de">Nachrichten des Tages</desc>
    <category lang="de">Nachrichten</category>
    <icon src="https://images.zattic.com/cms/.../original.jpg" />
  </programme>
</tv>
```

## Performance Optimization

### Recommended Settings for Best Performance

**For maximum speed (recommended):**
```bash
python zattoo_epg.py --no-details
```
- Only loads basic EPG data (title, time, description)
- **Very fast**: ~1-2 seconds per day
- Sufficient for most EPG viewers

**For detailed information (slow):**
```bash
python zattoo_epg.py
```
- Loads additional details (actors, directors, episode information)
- **Slow**: 20-30 minutes per day
- May be interrupted by rate limiting

### Performance Issues

Loading detailed information is very slow because:
- Each program requires a separate API call
- For 7 days this can be 80,000+ API requests
- Zattoo's API has rate limiting and blocks too many requests

**Solution**: Use `--no-details` for normal usage.

## TVHeadend Integration

### Automatic Sending to TVHeadend

The program can send generated EPG data directly to TVHeadend:

```bash
# Create EPG and send to TVHeadend
python zattoo_epg.py --no-details --tvheadend

# Send directly to TVHeadend without creating file
python zattoo_epg.py --no-details --tvheadend-only
```

### TVHeadend Configuration

1. **Enable XMLTV Grabber**:
   - In TVHeadend: Configuration → Channel/EPG → EPG Grabber
   - Enable "XMLTV"
   - Socket path: `/var/lib/tvheadend/epggrab/xmltv.sock`

2. **Check permissions**:
   ```bash
   # Check socket path
   ls -la /var/lib/tvheadend/epggrab/xmltv.sock
   
   # Add user to tvheadend group (if necessary)
   sudo usermod -a -G tvheadend $USER
   ```

3. **Automation with Cron**:
   ```bash
   # Update EPG daily at 6:00 AM
   0 6 * * * /path/to/python /path/to/zattoo_epg.py --no-details --tvheadend-only
   ```

### TVHeadend Troubleshooting

- **Socket not found**: TVHeadend is not running or XMLTV grabber is not enabled
- **Permission denied**: User does not have permission to access the socket
- **Connection failed**: TVHeadend XMLTV grabber is not configured

## Troubleshooting

### Common Issues

1. **Login failed**
   - Check your Zattoo credentials
   - Ensure your account is valid for the selected country

2. **Network errors**
   - Check your internet connection
   - Zattoo service may be temporarily unavailable

3. **Empty EPG file**
   - Possibly no program data available for the selected time period
   - Try a shorter time period

### Debug Information

The program outputs detailed progress information:
- Session token status
- Login status
- Number of loaded channels
- Download progress
- Number of processed programs

## License

This program is licensed under the GNU General Public License v3.0, just like the original easyEPG project.

## Contributors

Based on the original easyEPG project by:
- Jan-Luca Neumann (sunsettrack4)
- DeBaschdi

Python implementation developed as an equivalent alternative to the original Bash/Perl script.
