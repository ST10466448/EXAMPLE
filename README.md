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

        // COLLECTION
        Dictionary<string, List<string>> responses =
            new Dictionary<string, List<string>>()
        {
            {
                "password",
                new List<string>()
                {
                    "Use strong passwords with symbols and numbers.",
                    "Avoid using personal information in passwords.",
                    "Use different passwords for every account."
                }
            },

            {
                "phishing",
                new List<string>()
                {
                    "Never click suspicious email links.",
                    "Check the sender email carefully.",
                    "Phishing scams pretend to be trusted companies."
                }
            },

            {
                "malware",
                new List<string>()
                {
                    "Install antivirus software.",
                    "Avoid unsafe downloads.",
                    "Keep your computer updated."
                }
            },

            {
                "vpn",
                new List<string>()
                {
                    "VPNs protect your online privacy.",
                    "Use a VPN on public Wi-Fi.",
                    "VPNs encrypt internet traffic."
                }
            },

            {
                "privacy",
                new List<string>()
                {
                    "Review your privacy settings regularly.",
                    "Do not overshare online.",
                    "Enable two-factor authentication."
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
                DisplayBotMessage("It is understandable to feel worried.");

                DisplayBotMessage("Remember to use strong passwords and avoid suspicious links.");

                return;
            }

            if (input.Contains("frustrated") || input.Contains("angry"))
            {
                DisplayBotMessage("I understand your frustration.");

                return;
            }

            if (input.Contains("curious"))
            {
                DisplayBotMessage("Curiosity is great for learning cybersecurity.");

                return;
            }

            // CONVERSATION FLOW
            if (input.Contains("tell me more") || input.Contains("another tip"))
            {
                if (!string.IsNullOrEmpty(favouriteTopic))
                {
                    DisplayBotMessage($"Since you like {favouriteTopic}, remember to keep your accounts secure.");
                }
                else
                {
                    DisplayBotMessage("Always keep your software updated.");
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
            return "I did not understand that. Ask me about passwords, phishing, malware, VPNs, or privacy.";
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
