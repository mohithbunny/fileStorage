const STORAGE_KEY = "fileStorageUploads";

const appConfig = {
  supabaseUrl: "https://your-project.supabase.co",
  supabaseAnonKey: "your-anon-key",
  bucketName: "files",
};

const { createClient } = window.supabase;
const supabase = createClient(appConfig.supabaseUrl, appConfig.supabaseAnonKey);

const form = document.getElementById("upload-form");
const fileInput = document.getElementById("file-input");
const customNameInput = document.getElementById("custom-name");
const statusBox = document.getElementById("status");
const uploadList = document.getElementById("upload-list");
const clearHistoryButton = document.getElementById("clear-history");

function setStatus(message, type = "") {
  statusBox.textContent = message;
  statusBox.classList.remove("success", "error");

  if (type) {
    statusBox.classList.add(type);
  }
}

function formatBytes(bytes) {
  if (!Number.isFinite(bytes) || bytes <= 0) return "0 B";

  const units = ["B", "KB", "MB", "GB"];
  let value = bytes;
  let unitIndex = 0;

  while (value >= 1024 && unitIndex < units.length - 1) {
    value /= 1024;
    unitIndex += 1;
  }

  return `${value.toFixed(value >= 10 || unitIndex === 0 ? 0 : 1)} ${units[unitIndex]}`;
}

function isConfigured() {
  return (
    appConfig.supabaseUrl !== "https://your-project.supabase.co" &&
    appConfig.supabaseAnonKey !== "your-anon-key" &&
    appConfig.bucketName.trim().length > 0
  );
}

function readHistory() {
  try {
    const raw = localStorage.getItem(STORAGE_KEY);
    return raw ? JSON.parse(raw) : [];
  } catch {
    return [];
  }
}

function writeHistory(items) {
  localStorage.setItem(STORAGE_KEY, JSON.stringify(items));
}

function renderHistory() {
  const items = readHistory();

  if (!items.length) {
    uploadList.innerHTML = '<div class="empty-state">No files uploaded yet.</div>';
    return;
  }

  uploadList.innerHTML = items
    .slice()
    .reverse()
    .map(
      (item) => `
        <div class="upload-item">
          <div class="upload-meta">
            <span class="upload-name">${item.name}</span>
            <span class="upload-size">${item.size} • ${item.uploadedAt}</span>
          </div>
          <button class="copy-button" data-copy="${item.url}" type="button">Copy link</button>
          <a class="open-button" href="${item.url}" target="_blank" rel="noreferrer">Open</a>
        </div>
      `
    )
    .join("");

  document.querySelectorAll("[data-copy]").forEach((button) => {
    button.addEventListener("click", async () => {
      const url = button.getAttribute("data-copy");
      await navigator.clipboard.writeText(url);
      button.textContent = "Copied";
      setTimeout(() => {
        button.textContent = "Copy link";
      }, 1200);
    });
  });
}

async function uploadFile(file) {
  const fileName = customNameInput.value.trim() || file.name;
  const uniqueFileName = `${crypto.randomUUID()}-${fileName.replace(/\s+/g, "-")}`;

  const { data, error } = await supabase.storage.from(appConfig.bucketName).upload(uniqueFileName, file, {
    cacheControl: "3600",
    upsert: false,
    contentType: file.type || "application/octet-stream",
  });

  if (error) {
    throw new Error(error.message);
  }

  const { data: publicData } = supabase.storage.from(appConfig.bucketName).getPublicUrl(data.path);
  const publicUrl = publicData.publicUrl;

  const history = readHistory();
  const item = {
    name: fileName,
    size: formatBytes(file.size),
    url: publicUrl,
    uploadedAt: new Date().toLocaleString(),
  };

  history.push(item);
  writeHistory(history);
  renderHistory();

  return publicUrl;
}

form.addEventListener("submit", async (event) => {
  event.preventDefault();

  if (!isConfigured()) {
    setStatus(
      "Add your Supabase URL, anon key, and bucket name in app.js before uploading files.",
      "error"
    );
    return;
  }

  const selectedFile = fileInput.files[0];

  if (!selectedFile) {
    setStatus("Please choose a file before uploading.", "error");
    return;
  }

  const uploadButton = document.getElementById("upload-button");
  uploadButton.disabled = true;
  uploadButton.textContent = "Uploading...";
  setStatus("Uploading your file...");

  try {
    const uploadedUrl = await uploadFile(selectedFile);
    setStatus(`Upload complete. Share this link: ${uploadedUrl}`, "success");
    form.reset();
    customNameInput.value = "";
  } catch (error) {
    setStatus(`Upload failed: ${error.message}`, "error");
  } finally {
    uploadButton.disabled = false;
    uploadButton.textContent = "Upload file";
  }
});

clearHistoryButton.addEventListener("click", () => {
  localStorage.removeItem(STORAGE_KEY);
  renderHistory();
});

renderHistory();

if (!isConfigured()) {
  setStatus("Configure Supabase in app.js before using the uploader.");
}
