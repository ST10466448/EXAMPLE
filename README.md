using System;
using System.Collections.Generic;
using System.Media;
using System.Windows;

namespace POEPART2YEAR2
{
    public partial class MainWindow : Window
    {
        // MEMORY VARIABLES
        private string userName = "";
        private string favouriteTopic = "";

        // RANDOM OBJECT
        Random random = new Random();

        // DELEGATE
        delegate string BotResponse(string input);

        BotResponse responseDelegate;

        // COLLECTION WITH MORE TOPICS + DETAILED RESPONSES
        Dictionary<string, List<string>> responses =
            new Dictionary<string, List<string>>()
        {
            {
                "password",
                new List<string>()
                {
                    "Strong passwords should contain uppercase letters, lowercase letters, numbers, and symbols. Avoid using easy information like birthdays or names because hackers can guess them easily.",

                    "Using the same password for multiple accounts is risky. If one account is hacked, attackers can gain access to all your other accounts as well.",

                    "A password manager can help you generate and store secure passwords safely so that you do not need to remember every password yourself."
                }
            },

            {
                "phishing",
                new List<string>()
                {
                    "Phishing is a cyberattack where criminals pretend to be trusted organisations to trick people into giving away sensitive information such as passwords or banking details.",

                    "Always check email addresses carefully before clicking links. Many phishing emails look real but contain small spelling mistakes or suspicious domains.",

                    "If an email creates urgency such as 'Your account will be locked immediately', it could be a phishing attempt designed to pressure you into acting quickly."
                }
            },

            {
                "malware",
                new List<string>()
                {
                    "Malware is malicious software designed to damage systems, steal data, or spy on users. Examples include viruses, worms, ransomware, and spyware.",

                    "Avoid downloading files from unknown websites because malware is often hidden inside fake software, pirated programs, or suspicious attachments.",

                    "Keeping your operating system and antivirus software updated helps protect your device against newly discovered malware threats."
                }
            },

            {
                "vpn",
                new List<string>()
                {
                    "A VPN, or Virtual Private Network, encrypts your internet traffic and protects your privacy when browsing online.",

                    "Using public Wi-Fi without a VPN can expose your personal information to hackers who may intercept your internet traffic.",

                    "VPNs can also help prevent websites and advertisers from tracking your online activities and collecting your browsing data."
                }
            },

            {
                "privacy",
                new List<string>()
                {
                    "Protecting your privacy online means controlling who can access your personal information and how it is shared.",

                    "Avoid posting sensitive information such as your address, passwords, banking details, or personal documents online.",

                    "Always review privacy settings on social media platforms to control who can view your content and personal details."
                }
            },

            {
                "firewall",
                new List<string>()
                {
                    "A firewall acts as a security barrier between your computer and the internet by blocking suspicious or unauthorised traffic.",

                    "Firewalls help prevent hackers from gaining unauthorised access to your device or network.",

                    "Both hardware and software firewalls are important for protecting systems against cyber threats."
                }
            },

            {
                "antivirus",
                new List<string>()
                {
                    "Antivirus software scans your computer for malicious programs and helps remove threats before they can cause damage.",

                    "Regular antivirus scans are important because cyber threats are constantly evolving and new malware appears every day.",

                    "Keeping your antivirus updated ensures that it can detect the latest cyber threats effectively."
                }
            },

            {
                "social engineering",
                new List<string>()
                {
                    "Social engineering is when attackers manipulate people into revealing confidential information rather than hacking systems directly.",

                    "Cybercriminals often pretend to be trusted individuals such as IT staff, banks, or company employees to gain your trust.",

                    "Always verify a person's identity before sharing passwords, OTPs, or sensitive company information."
                }
            },

            {
                "ransomware",
                new List<string>()
                {
                    "Ransomware is a type of malware that locks or encrypts files and demands payment to restore access.",

                    "Backing up important files regularly can help you recover data without paying cybercriminals.",

                    "Never open suspicious email attachments because ransomware is commonly spread through phishing emails."
                }
            },

            {
                "2fa",
                new List<string>()
                {
                    "Two-factor authentication adds an extra layer of security by requiring a second verification step in addition to your password.",

                    "Even if hackers steal your password, two-factor authentication can still help prevent unauthorised access to your account.",

                    "Authentication apps are usually safer than SMS codes because text messages can sometimes be intercepted."
                }
            },

            {
                "scam",
                new List<string>()
                {
                    "Online scams are designed to trick users into sending money or revealing personal information.",

                    "Be cautious of messages promising prizes, quick money, or urgent requests because these are common scam tactics.",

                    "If something online seems too good to be true, it is usually a scam."
                }
            },

            {
                "cyberbullying",
                new List<string>()
                {
                    "Cyberbullying involves using technology to harass, threaten, or embarrass another person online.",

                    "If you experience cyberbullying, avoid responding to harmful messages and report the behaviour to the platform.",

                    "Saving screenshots of harmful messages can help provide evidence when reporting cyberbullying incidents."
                }
            },

            {
                "data breach",
                new List<string>()
                {
                    "A data breach occurs when sensitive information is accessed, stolen, or exposed without authorisation.",

                    "Companies should encrypt sensitive data and implement strong access controls to reduce the risk of data breaches.",

                    "If you are notified about a data breach, change your passwords immediately and monitor your accounts for suspicious activity."
                }
            },

            {
                "safe browsing",
                new List<string>()
                {
                    "Safe browsing involves visiting trusted websites and avoiding suspicious links or downloads.",

                    "Always look for HTTPS in the website address because it indicates that the connection is encrypted.",

                    "Avoid clicking pop-up advertisements or downloading files from untrusted websites because they may contain malware."
                }
            }
        };

        public MainWindow()
        {
            InitializeComponent();

            DisplayBotMessage("Hello! I am your Cybersecurity Awareness Bot.");
            DisplayBotMessage("What is your name?");

            PlayGreeting();
        }

        private void SendButton_Click(object sender, RoutedEventArgs e)
        {
            string input = UserInput.Text.Trim();

            // ERROR HANDLING
            if (string.IsNullOrWhiteSpace(input))
            {
                DisplayBotMessage("Please enter a message.");
                return;
            }

            DisplayUserMessage(input);

            HandleConversation(input.ToLower());

            UserInput.Clear();
        }

        private void HandleConversation(string input)
        {
            // STORE USER NAME
            if (string.IsNullOrEmpty(userName))
            {
                userName = input;

                DisplayBotMessage($"Nice to meet you, {userName}!");

                return;
            }

            // MEMORY FEATURE
            if (input.Contains("interested in"))
            {
                favouriteTopic = input.Replace("interested in", "").Trim();

                DisplayBotMessage($"I will remember that you are interested in {favouriteTopic}.");

                return;
            }

            // SENTIMENT DETECTION
            if (input.Contains("worried") || input.Contains("scared"))
            {
                DisplayBotMessage("It is understandable to feel worried about cybersecurity threats.");

                DisplayBotMessage("Remember to use strong passwords, avoid suspicious links, and keep your software updated.");

                return;
            }

            if (input.Contains("frustrated") || input.Contains("angry"))
            {
                DisplayBotMessage("I understand your frustration. Cybersecurity can sometimes feel overwhelming, but learning safe practices helps reduce risks.");

                return;
            }

            if (input.Contains("curious"))
            {
                DisplayBotMessage("Curiosity is great for learning cybersecurity and staying informed about online safety.");

                return;
            }

            // CONVERSATION FLOW
            if (input.Contains("tell me more") || input.Contains("another tip"))
            {
                if (!string.IsNullOrEmpty(favouriteTopic))
                {
                    DisplayBotMessage($"Since you are interested in {favouriteTopic}, remember to stay informed and use secure online practices.");
                }
                else
                {
                    DisplayBotMessage("Always keep your software updated and avoid clicking suspicious links or downloading unknown files.");
                }

                return;
            }

            // KEYWORD RECOGNITION
            foreach (var keyword in responses.Keys)
            {
                if (input.Contains(keyword))
                {
                    List<string> possibleResponses = responses[keyword];

                    int index = random.Next(possibleResponses.Count);

                    string selectedResponse = possibleResponses[index];

                    DisplayBotMessage(selectedResponse);

                    return;
                }
            }

            // DELEGATE USAGE
            responseDelegate = GetHelpResponse;

            DisplayBotMessage(responseDelegate(input));
        }

        private string GetHelpResponse(string input)
        {
            return "I did not understand that. You can ask me about passwords, phishing, malware, VPNs, privacy, ransomware, antivirus, social engineering, scams, firewalls, safe browsing, or two-factor authentication.";
        }

        private void DisplayUserMessage(string message)
        {
            ChatDisplay.AppendText($"YOU: {message}\n\n");
        }

        private void DisplayBotMessage(string message)
        {
            ChatDisplay.AppendText($"BOT: {message}\n\n");

            ChatDisplay.ScrollToEnd();
        }

        private void PlayGreeting()
        {
            try
            {
                SoundPlayer player = new SoundPlayer("greeting.wav");

                player.Play();
            }
            catch
            {
                DisplayBotMessage("Voice greeting unavailable.");
            }
        }
    }
}
           <Window x:Class="POEPART2YEAR2.MainWindow"
        xmlns="http://schemas.microsoft.com/winfx/2006/xaml/presentation"
        xmlns:x="http://schemas.microsoft.com/winfx/2006/xaml"
        Title="Cybersecurity Awareness Bot"
        Height="650"
        Width="1000"
        WindowStartupLocation="CenterScreen"
        Background="#121212"
        ResizeMode="CanMinimize">

    <Grid Margin="15">

        <!-- ROWS -->
        <Grid.RowDefinitions>
            <RowDefinition Height="Auto"/>
            <RowDefinition Height="*"/>
            <RowDefinition Height="Auto"/>
        </Grid.RowDefinitions>

        <!-- HEADER -->
        <Border Grid.Row="0"
                Background="#00A8E8"
                CornerRadius="20"
                Padding="20"
                Margin="0,0,0,15">

            <StackPanel>

                <TextBlock Text="🔒 CYBERSECURITY AWARENESS BOT 🔒"
                           FontSize="30"
                           FontWeight="Bold"
                           Foreground="White"
                           HorizontalAlignment="Center"/>

                <TextBlock Text="Stay Safe Online • Protect Your Privacy • Learn Cybersecurity"
                           FontSize="15"
                           Margin="0,10,0,0"
                           Foreground="White"
                           HorizontalAlignment="Center"/>

            </StackPanel>

        </Border>

        <!-- CHAT SECTION -->
        <Border Grid.Row="1"
                Background="#1E1E1E"
                CornerRadius="20"
                Padding="10"
                BorderBrush="#00A8E8"
                BorderThickness="2">

            <ScrollViewer VerticalScrollBarVisibility="Auto">

                <TextBox x:Name="ChatDisplay"
                         Background="#1E1E1E"
                         Foreground="White"
                         FontSize="16"
                         FontFamily="Consolas"
                         IsReadOnly="True"
                         TextWrapping="Wrap"
                         AcceptsReturn="True"
                         BorderThickness="0"
                         VerticalScrollBarVisibility="Hidden"
                         Padding="10"/>

            </ScrollViewer>

        </Border>

        <!-- INPUT SECTION -->
        <Border Grid.Row="2"
                Background="#1E1E1E"
                CornerRadius="20"
                Padding="10"
                Margin="0,15,0,0">

            <Grid>

                <Grid.ColumnDefinitions>
                    <ColumnDefinition Width="*"/>
                    <ColumnDefinition Width="140"/>
                </Grid.ColumnDefinitions>

                <!-- USER INPUT -->
                <TextBox x:Name="UserInput"
                         Height="45"
                         FontSize="15"
                         FontFamily="Segoe UI"
                         VerticalContentAlignment="Center"
                         Padding="15,0,15,0"
                         Background="#2D2D30"
                         Foreground="White"
                         BorderBrush="#00A8E8"
                         BorderThickness="2"
                         CaretBrush="White"/>

                <!-- SEND BUTTON -->
                <Button Grid.Column="1"
                        Content="SEND"
                        Margin="15,0,0,0"
                        FontSize="16"
                        FontWeight="Bold"
                        Background="#00A8E8"
                        Foreground="White"
                        BorderThickness="0"
                        Cursor="Hand"
                        Click="SendButton_Click">

                    <Button.Style>

                        <Style TargetType="Button">

                            <Setter Property="Template">

                                <Setter.Value>

                                    <ControlTemplate TargetType="Button">

                                        <Border Background="{TemplateBinding Background}"
                                                CornerRadius="15">

                                            <ContentPresenter HorizontalAlignment="Center"
                                                              VerticalAlignment="Center"/>

                                        </Border>

                                    </ControlTemplate>

                                </Setter.Value>

                            </Setter>

                        </Style>

                    </Button.Style>

                </Button>

            </Grid>

        </Border>

    </Grid>

</Window>
