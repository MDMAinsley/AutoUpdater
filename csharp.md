using Newtonsoft.Json;

namespace Test
{
    public partial class Main : Form
    {
        public Main()
        {
            InitializeComponent();
            // _ = CheckForUpdates();
        }
        public async Task CheckForUpdates()
        {
            string release = UpdateManager.GitHubHelper.GetLatestReleaseVersion("MDMAinsley", "AutoUpdater");
            if (release != lblProgramVersion.Text)
            {
                DialogResult result = MessageBox.Show($"Update: {release} is available, would you like to download it?", "Update Available", MessageBoxButtons.YesNo, MessageBoxIcon.Information);
                if (result == DialogResult.Yes)
                {
                    UpdateManager updateManager = new UpdateManager("https://www.github.com/MDMAinsley/AutoUpdater/releases/latest/download/README.md", @"C:\Users\lewis\Documents\AutoUpdaterTest\README.md");
                    await updateManager.DownloadUpdateAsync();

                    lblProgramVersion.Text = release;
                }                                        
            }
        }
        private void llCheckForUpdates_Click(object sender, EventArgs e)
        {
            _ = CheckForUpdates();
        }        
    }
    public class UpdateManager
    {
        public class GitHubRelease
        {
            public string tag_name { get; set; }
            public string html_url { get; set; }
        }

        public class GitHubHelper
        {
            public static string GetLatestReleaseVersion(string repositoryOwner, string repositoryName)
            {
                using (var client = new HttpClient())
                {
                    string apiUrl = $"https://api.github.com/repos/{repositoryOwner}/{repositoryName}/releases/latest";
                    client.DefaultRequestHeaders.Add("User-Agent", "request");
                    var response = client.GetStringAsync(apiUrl);
                    var release = JsonConvert.DeserializeObject<GitHubRelease>(response.Result);
                    return release.tag_name;
                }
            }
        }

        private readonly string downloadUrl;
        private readonly string localFilePath;

        public UpdateManager(string downloadUrl, string localFilePath)
        {
            this.downloadUrl = downloadUrl;
            this.localFilePath = localFilePath;
        }

        public async Task DownloadUpdateAsync()
        {
            try
            {
                using (HttpClient client = new HttpClient())
                {
                    using (HttpResponseMessage response = await client.GetAsync(downloadUrl, HttpCompletionOption.ResponseHeadersRead))
                    {
                        response.EnsureSuccessStatusCode(); // Throws if HTTP response status code is not a success code

                        using (Stream contentStream = await response.Content.ReadAsStreamAsync(),
                                     fileStream = new FileStream(localFilePath, FileMode.Create, FileAccess.Write, FileShare.None, bufferSize: 8192, useAsync: true))
                        {
                            await contentStream.CopyToAsync(fileStream);
                        }
                    }
                }

                MessageBox.Show("Update downloaded successfully!", "Success", MessageBoxButtons.OK, MessageBoxIcon.Information);
                // Install the update -- Obselete for now
                // InstallUpdate();
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Failed to download update: {ex.Message}", "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }

        private void InstallUpdate()
        {
            try
            {
                // Replace the old file with the new one
                File.Copy(localFilePath, localFilePath, true);
                MessageBox.Show("Update installed successfully! Please restart the application.", "Success", MessageBoxButtons.OK, MessageBoxIcon.Information);
            }
            catch (Exception ex)
            {
                MessageBox.Show($"Failed to install update: {ex.Message}", "Error", MessageBoxButtons.OK, MessageBoxIcon.Error);
            }
        }
    }
}