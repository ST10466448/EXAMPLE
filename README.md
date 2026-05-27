using System;
using System.Collections.Generic;
using System.Linq;
using System.Media;
using System.Threading.Tasks;
using System.Windows;
using System.Windows.Controls;
using System.Windows.Input;
using System.Windows.Media;

namespace CyberBot
{
    public partial class MainWindow : Window
    {
        // ─── MEMORY ───────────────────────────────────────────────────────────
        private string _userName  = null;
        private string _favTopic  = null;
        private string _lastTopic = null;

        private static readonly Random _rng = new Random();

        // ─── RANDOM RESPONSES ─────────────────────────────────────────────────
        private static readonly Dictionary<string, string[]> Responses = new()
        {
            ["password"] = new[]
            {
                "Strong passwords are your first line of defence. Use 12+ characters mixing uppercase, lowercase, numbers, and symbols. Never reuse passwords across accounts.",
                "Consider a passphrase — a string of random words like 'correct horse battery staple'. It's both memorable and very hard to crack.",
                "A password manager like Bitwarden or 1Password can generate and store complex unique passwords for every site. Highly recommended!"
            },
            ["phishing"] = new[]
            {
                "Phishing emails often impersonate trusted brands. Look for mismatched sender addresses, urgency tactics, and suspicious links. Hover before you click.",
                "Be cautious of emails asking for personal information. Scammers disguise themselves as trusted organisations — banks, tech companies, even the government.",
                "Check for HTTPS and the exact domain name before entering credentials. Attackers use lookalike domains like 'paypa1.com' or 'g00gle.com'."
            },
            ["malware"] = new[]
            {
                "Malware can arrive via email attachments, infected downloads, or malicious ads. Keep your OS and antivirus updated and scan regularly.",
                "Ransomware encrypts your files and demands payment. Back up your data regularly to an offline location so you're never held hostage.",
                "Avoid downloading software from unofficial sources. Stick to official app stores and vendor websites to reduce infection risk."
            },
            ["vpn"] = new[]
            {
                "A VPN encrypts your traffic and masks your IP address, keeping you private on public Wi-Fi and from your ISP.",
                "Not all VPNs are equal — avoid free ones that may log and sell your data. Look for audited, no-log providers like Mullvad or ProtonVPN.",
                "A VPN helps bypass geo-restrictions, but it's not a silver bullet — you still need strong passwords and updated software."
            },
            ["privacy"] = new[]
            {
                "Digital privacy starts with reviewing app permissions. Does that flashlight app really need your contacts and location?",
                "Use privacy-focused tools: DuckDuckGo for search, Signal for messaging, Firefox with uBlock Origin for browsing.",
                "Enable two-factor authentication everywhere you can. Even if your password leaks, 2FA keeps attackers out."
            },
            ["scam"] = new[]
            {
                "If an offer seems too good to be true online, it almost certainly is. Verify before you click, share, or pay.",
                "Romance scams target victims emotionally over weeks before requesting money. Be cautious of online relationships that never meet in person.",
                "Tech support scams often appear as pop-ups claiming your device is infected. Legitimate companies don't cold-call you about viruses."
            },
            ["safebrowsing"] = new[]
            {
                "Always verify URLs begin with HTTPS and match the real domain. Use a browser extension like uBlock Origin to block malicious ads.",
                "Keep your browser and extensions updated. Many exploits target outdated plugins — disable any you don't actively use.",
                "Public Wi-Fi is dangerous without a VPN. Attackers can intercept unencrypted traffic. Avoid banking on public networks."
            },
            ["2fa"] = new[]
            {
                "Two-factor authentication (2FA) adds a second verification step beyond your password — a code from an app or SMS. Enable it everywhere.",
                "Use an authenticator app like Google Authenticator or Aegis rather than SMS for 2FA — SIM-swap attacks can intercept text codes.",
                "Hardware security keys like YubiKey are the gold standard for 2FA — nearly impossible to phish remotely."
            }
        };

        // ─── SENTIMENT KEYWORDS ───────────────────────────────────────────────
        private static readonly Dictionary<string, string[]> SentimentKeywords = new()
        {
            ["worried"]    = new[] { "worried", "scared", "afraid", "anxious", "nervous", "unsafe", "terrified", "concerned", "stress" },
            ["frustrated"] = new[] { "frustrated", "annoyed", "angry", "hate", "terrible", "awful", "useless", "stupid", "mad" },
            ["curious"]    = new[] { "curious", "interested", "wonder", "how", "what", "explain", "tell me", "learn", "understand" }
        };

        private static readonly Dictionary<string, string> SentimentPrefixes = new()
        {
            ["worried"]    = "It's completely understandable to feel that way. With the right knowledge you can stay protected. Here's what I know:\n\n",
            ["frustrated"] = "I hear you — cybersecurity can feel overwhelming. Let's break it down simply:\n\n",
            ["curious"]    = "Great question! Understanding this is the best way to stay safe. Here's the breakdown:\n\n",
            ["neutral"]    = ""
        };

        private static readonly Dictionary<string, (string Label, string Bg, string Fg)> SentimentStyles = new()
        {
            ["worried"]    = ("EMPATHY MODE",  "#1A0008", "#FF4466"),
            ["frustrated"] = ("CALM MODE",     "#1A0800", "#FFAA00"),
            ["curious"]    = ("EXPLORE MODE",  "#001A0E", "#00FF88"),
        };

        // ─── CONSTRUCTOR ──────────────────────────────────────────────────────
        public MainWindow()
        {
            InitializeComponent();
            Loaded += (_, _) =>
            {
                PlayGreeting();
                ShowWelcomeBubble();
                UserInputBox.Focus();
            };
        }

        // ─── VOICE GREETING ───────────────────────────────────────────────────
        private static void PlayGreeting()
        {
            try { new SoundPlayer("greeting.wav").Play(); }
            catch { /* greeting.wav not found — silent fail */ }
        }

        // ─── SEND / ENTER ─────────────────────────────────────────────────────
        private void SendButton_Click(object sender, RoutedEventArgs e) => ProcessInput();
        private void UserInputBox_KeyDown(object sender, KeyEventArgs e)
        {
            if (e.Key == Key.Enter) ProcessInput();
        }

        // ─── MAIN PROCESSOR ───────────────────────────────────────────────────
        private async void ProcessInput()
        {
            string input = UserInputBox.Text.Trim();
            if (string.IsNullOrWhiteSpace(input)) return;

            UserInputBox.Clear();
            AppendUserBubble(input);

            SendButton.IsEnabled   = false;
            UserInputBox.IsEnabled = false;

            await Task.Delay(600 + _rng.Next(400));

            string sentiment = DetectSentiment(input.ToLower());
            string response  = BuildResponse(input.ToLower(), sentiment);

            AppendBotBubble(response, sentiment);
            UpdateMemoryBar();

            SendButton.IsEnabled   = true;
            UserInputBox.IsEnabled = true;
            UserInputBox.Focus();
        }

        // ─── RESPONSE BUILDER ─────────────────────────────────────────────────
        private string BuildResponse(string lower, string sentiment)
        {
            // Name capture
            foreach (var phrase in new[] { "my name is ", "i am ", "i'm ", "call me " })
            {
                int idx = lower.IndexOf(phrase);
                if (idx >= 0)
                {
                    string captured = lower[(idx + phrase.Length)..].Split(' ')[0].Trim();
                    if (captured.Length > 1 && !new[] { "hi", "ok", "yes", "no", "hey", "bye", "not" }.Contains(captured))
                    {
                        _userName = char.ToUpper(captured[0]) + captured[1..];
                        return $"Nice to meet you, {_userName}! I'll remember your name. Ask me anything about cybersecurity — passwords, phishing, malware, VPNs, privacy, or scams.";
                    }
                }
            }

            // Farewell
            if (lower.Contains("bye") || lower.Contains("exit") || lower.Contains("quit") || lower.Contains("goodbye"))
            {
                string name = _userName != null ? $", {_userName}" : "";
                return $"Stay safe out there{name}! Remember: strong passwords, think before you click, and always verify. Goodbye! 🔒";
            }

            // Greetings
            if (IsGreeting(lower))
            {
                string name = _userName != null ? $", {_userName}" : "";
                return $"Hello{name}! I'm Cyber Sentinel, your cybersecurity guide. What would you like to learn about today?";
            }

            // How are you
            if (lower.Contains("how are you") || lower.Contains("how're you"))
                return "All systems operational! My threat sensors are active and I'm ready to help you stay safe online. What cybersecurity topic can I assist you with?";

            // Purpose
            if (lower.Contains("purpose") || lower.Contains("what do you do") || lower.Contains("what can you do"))
                return "My purpose is to educate users about cybersecurity risks and best practices. I cover passwords, phishing, malware, VPNs, privacy, scams, safe browsing, and two-factor authentication.";

            // Favourite topic memory
            foreach (var phrase in new[] { "interested in ", "i like ", "i love ", "my favourite ", "i prefer " })
            {
                int idx = lower.IndexOf(phrase);
                if (idx >= 0)
                {
                    string topic = lower[(idx + phrase.Length)..].Trim();
                    _favTopic = topic;
                    return $"Great! I'll remember that you're interested in {topic}. It's a crucial part of staying safe online. I'll tailor my tips with that in mind!";
                }
            }

            // Follow-up / conversation flow
            if (lower.Contains("tell me more") || lower.Contains("explain more") ||
                lower.Contains("give me another") || lower.Contains("more info")  ||
                lower.Contains("elaborate")      || lower.Contains("continue")    ||
                lower.Contains("another tip"))
            {
                if (_lastTopic != null && Responses.ContainsKey(_lastTopic))
                    return $"Here's another angle on {_lastTopic}:\n\n{Rand(Responses[_lastTopic])}";
                return "Could you let me know which topic you'd like more on? I cover passwords, phishing, malware, VPNs, privacy, and scams.";
            }

            // Topic detection + random response selection
            string detectedTopic = DetectTopic(lower);
            if (detectedTopic != null)
            {
                _lastTopic = detectedTopic;
                string prefix = (_favTopic != null && lower.Contains(_favTopic))
                    ? $"As someone interested in {_favTopic}, here's an important tip:\n\n" : "";
                string sentimentPrefix = SentimentPrefixes.GetValueOrDefault(sentiment, "");
                return sentimentPrefix + prefix + Rand(Responses[detectedTopic]);
            }

            // Fallback
            string n = _userName != null ? $", {_userName}" : "";
            return $"I didn't quite understand that{n}. You can ask me about:\n\nPasswords · Phishing · Malware · VPN · Privacy · Scams · Safe Browsing · 2FA\n\nOr say \"tell me more\" to continue the last topic.";
        }

        // ─── TOPIC DETECTION ──────────────────────────────────────────────────
        private static string DetectTopic(string lower)
        {
            if (lower.Contains("password") || lower.Contains("passphrase"))             return "password";
            if (lower.Contains("phish")    || lower.Contains("email scam"))             return "phishing";
            if (lower.Contains("malware")  || lower.Contains("virus") ||
                lower.Contains("ransomware")|| lower.Contains("trojan"))                return "malware";
            if (lower.Contains("vpn")      || lower.Contains("virtual private"))        return "vpn";
            if (lower.Contains("2fa")      || lower.Contains("two factor") ||
                lower.Contains("two-factor")|| lower.Contains("authenticat"))           return "2fa";
            if (lower.Contains("privacy"))                                               return "privacy";
            if (lower.Contains("scam")     || lower.Contains("fraud"))                  return "scam";
            if (lower.Contains("safe brows")|| lower.Contains("https") ||
                lower.Contains("browsing") || lower.Contains("public wifi") ||
                lower.Contains("wi-fi"))                                                 return "safebrowsing";
            return null;
        }

        // ─── SENTIMENT DETECTION ──────────────────────────────────────────────
        private static string DetectSentiment(string lower)
        {
            foreach (var (sentiment, keywords) in SentimentKeywords)
                if (keywords.Any(k => lower.Contains(k)))
                    return sentiment;
            return "neutral";
        }

        private static bool IsGreeting(string lower)
        {
            var greets = new[] { "hi", "hello", "hey", "greetings", "good morning", "good afternoon", "good evening" };
            return greets.Any(g => lower.Trim() == g || lower.StartsWith(g + " ") || lower.StartsWith(g + "!"));
        }

        private static string Rand(string[] arr) => arr[_rng.Next(arr.Length)];

        // ─── UI: WELCOME BUBBLE ───────────────────────────────────────────────
        private void ShowWelcomeBubble()
        {
            var panel = new StackPanel();
            panel.Children.Add(new TextBlock
            {
                Text         = "Welcome! I'm Cyber Sentinel — your cybersecurity awareness guide.\nWhat would you like to explore?",
                Foreground   = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#C8D8E8")),
                FontSize     = 13,
                TextWrapping = TextWrapping.Wrap,
                Margin       = new Thickness(0, 0, 0, 10)
            });

            var buttonWrap = new WrapPanel { Orientation = Orientation.Horizontal };
            foreach (var topic in new[] { "Passwords", "Phishing", "Malware", "VPN", "Privacy", "Scams", "Safe Browsing", "2FA" })
            {
                var btn = new Button { Content = topic, Style = (Style)FindResource("TopicButton") };
                string captured = topic;
                btn.Click += (_, _) => { UserInputBox.Text = $"Tell me about {captured}"; ProcessInput(); };
                buttonWrap.Children.Add(btn);
            }
            panel.Children.Add(buttonWrap);
            AppendBotBubbleContent(panel, "neutral");
        }

        // ─── UI: USER BUBBLE ──────────────────────────────────────────────────
        private void AppendUserBubble(string text)
        {
            var bubble = new Border { Style = (Style)FindResource("UserBubble") };
            bubble.Child = new TextBlock
            {
                Text         = text,
                Foreground   = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#AAEEBB")),
                FontSize     = 13,
                TextWrapping = TextWrapping.Wrap
            };

            var row = new Grid();
            row.ColumnDefinitions.Add(new ColumnDefinition { Width = new GridLength(1, GridUnitType.Star) });
            row.ColumnDefinitions.Add(new ColumnDefinition { Width = GridLength.Auto });

            var av = MakeAvatar(_userName?[..1] ?? "U", "#001A0E", "#004422", "#00FF88");
            Grid.SetColumn(bubble, 0);
            Grid.SetColumn(av, 1);

            row.Children.Add(bubble);
            row.Children.Add(av);
            row.Margin = new Thickness(8, 4, 8, 4);
            ChatPanel.Children.Add(row);
            ChatScrollViewer.ScrollToEnd();
        }

        // ─── UI: BOT BUBBLE (string) ──────────────────────────────────────────
        private void AppendBotBubble(string text, string sentiment)
        {
            var content = new StackPanel();

            if (sentiment != "neutral" && SentimentStyles.TryGetValue(sentiment, out var style))
            {
                content.Children.Add(new Border
                {
                    Background      = new SolidColorBrush((Color)ColorConverter.ConvertFromString(style.Bg)),
                    BorderBrush     = new SolidColorBrush((Color)ColorConverter.ConvertFromString(style.Fg)),
                    BorderThickness = new Thickness(1),
                    CornerRadius    = new CornerRadius(3),
                    Padding         = new Thickness(6, 2, 6, 2),
                    Margin          = new Thickness(0, 0, 0, 6),
                    HorizontalAlignment = HorizontalAlignment.Left,
                    Child           = new TextBlock
                    {
                        Text       = style.Label,
                        Foreground = new SolidColorBrush((Color)ColorConverter.ConvertFromString(style.Fg)),
                        FontFamily = new FontFamily("Courier New"),
                        FontSize   = 10
                    }
                });
            }

            content.Children.Add(new TextBlock
            {
                Text         = text,
                Foreground   = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#C8D8E8")),
                FontSize     = 13,
                TextWrapping = TextWrapping.Wrap
            });

            AppendBotBubbleContent(content, sentiment);
        }

        // ─── UI: BOT BUBBLE (UIElement) ───────────────────────────────────────
        private void AppendBotBubbleContent(UIElement content, string sentiment)
        {
            var bubble = new Border { Style = (Style)FindResource("BotBubble") };
            bubble.Child = content;

            var av = MakeAvatar("CS", "#001A2E", "#00B8CC", "#00E5FF");

            var row = new Grid();
            row.ColumnDefinitions.Add(new ColumnDefinition { Width = GridLength.Auto });
            row.ColumnDefinitions.Add(new ColumnDefinition { Width = new GridLength(1, GridUnitType.Star) });

            Grid.SetColumn(av, 0);
            Grid.SetColumn(bubble, 1);
            row.Children.Add(av);
            row.Children.Add(bubble);
            row.Margin = new Thickness(8, 4, 8, 4);

            ChatPanel.Children.Add(row);
            ChatScrollViewer.ScrollToEnd();
        }

        // ─── UI: AVATAR ───────────────────────────────────────────────────────
        private static Border MakeAvatar(string initials, string bg, string border, string fg)
        {
            return new Border
            {
                Width           = 32, Height = 32,
                CornerRadius    = new CornerRadius(16),
                Background      = new SolidColorBrush((Color)ColorConverter.ConvertFromString(bg)),
                BorderBrush     = new SolidColorBrush((Color)ColorConverter.ConvertFromString(border)),
                BorderThickness = new Thickness(1),
                VerticalAlignment = VerticalAlignment.Top,
                Margin          = new Thickness(0, 4, 0, 0),
                Child           = new TextBlock
                {
                    Text                = initials,
                    Foreground          = new SolidColorBrush((Color)ColorConverter.ConvertFromString(fg)),
                    FontFamily          = new FontFamily("Courier New"),
                    FontSize            = 11,
                    FontWeight          = FontWeights.Bold,
                    HorizontalAlignment = HorizontalAlignment.Center,
                    VerticalAlignment   = VerticalAlignment.Center
                }
            };
        }

        // ─── UI: MEMORY BAR ───────────────────────────────────────────────────
        private void UpdateMemoryBar()
        {
            MemoryPanel.Children.Clear();
            MemoryPanel.Children.Add(new TextBlock
            {
                Text       = "MEM://",
                FontFamily = new FontFamily("Courier New"),
                FontSize   = 11,
                Foreground = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#334455")),
                VerticalAlignment = VerticalAlignment.Center,
                Margin     = new Thickness(0, 0, 8, 0)
            });

            var tags = new List<(string key, string val)>();
            if (_userName  != null) tags.Add(("name",     _userName));
            if (_favTopic  != null) tags.Add(("interest", _favTopic));
            if (_lastTopic != null) tags.Add(("last",     _lastTopic));

            if (tags.Count == 0)
            {
                MemoryPanel.Children.Add(new TextBlock
                {
                    Text       = "no data stored",
                    FontFamily = new FontFamily("Courier New"),
                    FontSize   = 11,
                    Foreground = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#334455")),
                    VerticalAlignment = VerticalAlignment.Center
                });
                return;
            }

            foreach (var (key, val) in tags)
            {
                MemoryPanel.Children.Add(new Border
                {
                    Background      = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#1A1400")),
                    BorderBrush     = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#332800")),
                    BorderThickness = new Thickness(1),
                    CornerRadius    = new CornerRadius(4),
                    Padding         = new Thickness(8, 2, 8, 2),
                    Margin          = new Thickness(0, 0, 6, 0),
                    Child           = new TextBlock
                    {
                        Text       = $"{key}:{val}",
                        FontFamily = new FontFamily("Courier New"),
                        FontSize   = 11,
                        Foreground = new SolidColorBrush((Color)ColorConverter.ConvertFromString("#FFAA00")),
                        VerticalAlignment = VerticalAlignment.Center
                    }
                });
            }
        }
    }
}
