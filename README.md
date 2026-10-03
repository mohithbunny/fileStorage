# FileStorage - Document Management Web App

A modern, user-friendly web application for uploading, storing, and accessing files (PDFs, Word docs, etc.) with unique shareable URLs.

## Features

✅ **Drag & Drop Upload** - Easy file upload with drag-and-drop interface
✅ **Multiple File Types** - Support for PDFs, Word docs (.docx, .doc), Excel sheets, images, and more
✅ **Unique URLs** - Each uploaded file gets a unique shareable URL
✅ **File Management** - View, download, and delete your uploaded files
✅ **Responsive Design** - Works on desktop, tablet, and mobile devices
✅ **GitHub Storage** - Files stored using GitHub's free storage infrastructure
✅ **Fast Access** - CDN-backed file delivery via GitHub Raw Content

## Tech Stack

- **Frontend**: HTML5, CSS3, JavaScript (Vanilla)
- **Backend**: Node.js + Express.js
- **Storage**: GitHub Repository (via GitHub API)
- **Deployment**: GitHub Pages (Frontend) + Vercel/Heroku (Backend)
- **Authentication**: GitHub Personal Access Token

## Project Structure

```
fileStorage/
├── index.html           # Main web app (Frontend)
├── css/
│   └── style.css       # Styles
├── js/
│   └── app.js          # Frontend logic
├── server/
│   ├── server.js       # Express backend
│   ├── package.json    # Dependencies
│   └── .env.example    # Environment variables template
└── docs/
    └── API.md          # API Documentation
```

## Setup Instructions

### 1. Backend Setup (Local/Deployment)

```bash
cd server
npm install
```

Create `.env` file:
```
GITHUB_TOKEN=your_github_personal_access_token
GITHUB_REPO_OWNER=mohithbunny
GITHUB_REPO_NAME=fileStorage-data
GITHUB_BRANCH=main
PORT=3000
```

### 2. Get GitHub Personal Access Token

1. Go to https://github.com/settings/tokens
2. Click "Generate new token"
3. Select scopes: `repo` (full control of private repositories)
4. Copy the token and add to `.env`

### 3. Create Storage Repository

Create a new private repository named `fileStorage-data` for storing uploaded files.

### 4. Deploy Backend

- **Option A**: Deploy to Vercel
  ```bash
  npm install -g vercel
  vercel
  ```

- **Option B**: Deploy to Heroku
  ```bash
  heroku create
  git push heroku main
  ```

### 5. Deploy Frontend to GitHub Pages

1. Update the API endpoint in `js/app.js` with your backend URL
2. Push to GitHub - the `gh-pages` branch will be auto-deployed

## How It Works

1. User selects/drags file to upload
2. Frontend sends file to backend API
3. Backend converts file to Base64 and commits to GitHub storage repo
4. GitHub generates a unique commit SHA
5. File is accessible via: `https://raw.githubusercontent.com/mohithbunny/fileStorage-data/main/{file-name}`
6. User gets a shareable unique URL

## API Endpoints

```
POST   /api/upload      - Upload a file
GET    /api/files       - List all uploaded files
GET    /api/file/:id    - Get file metadata
DELETE /api/file/:id    - Delete a file
```

## Security Notes

- Keep your GitHub token secure (use environment variables)
- Store token on backend only, never expose in frontend
- Consider rate limiting for public deployments
- Validate file types and sizes

## Free Storage Limits

- GitHub: 100GB per repository
- GitHub API: 60 requests/hour (unauthenticated), 5,000/hour (authenticated)

## License

MIT

## Support

For issues or questions, open an issue on this repository.
