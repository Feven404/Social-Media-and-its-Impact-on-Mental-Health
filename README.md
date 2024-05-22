# Social-Media-and-its-Impact-on-Mental-Health
# By: Feven Ferede and Ruth Girma  

# Introduction
This project explores the impact of social media use on mental health, focusing on time spent, preferred platforms, and content type. With rising mental health concerns and increased social media usage, especially among young adults, our research aims to uncover insights to inform better usage practices and interventions for improved mental well-being.

<div align = "center">
<img src = "images/intro image.jpg" width = "450")>
</div>

# Data cleaning
1. After importing the Excel and CSV files, we renamed the columns to make it easier to combine and analyze the data.
```
colnames(effect_social_media_response)[colnames(effect_social_media_response) == "Timestamp"] <- "Time"
colnames(social_media)[colnames(social_media) == "Timestamp"] <- "Time"
colnames(social_media)[colnames(social_media) == "X1..What.is.your.age."] <- "Age"
colnames(social_media)[colnames(social_media) == "X2..Gender"] <- "Gender"
colnames(social_media)[colnames(social_media) == "X3..Relationship.Status"] <- "Relationship"
colnames(social_media)[colnames(social_media) == "X4..Occupation.Status"] <- "Occupation"

```
2. Convert the Time columns in the datasets to character type. This ensures consistency in data type for easier manipulation and analysis.
```
combined_uber_data$Hour <- hour(combined_uber_data$Date.Time)
combined_uber_data$Month <- month.name[month(combined_uber_data$Date.Time)]
combined_uber_data$Day <- day(combined_uber_data$Date.Time)
combined_uber_data$Weekday <- weekdays(combined_uber_data$Date.Time)

```
3. We did sentiment analysis so we grouped similar sentiments under one so it would be easier to analyze the data
```
dplyr::mutate(Sentiment = ifelse(Sentiment == 'wonderment', 'wonder', Sentiment)) %>%
  dplyr::mutate(Sentiment = ifelse(Sentiment == 'thrilling journey', 'thrill', Sentiment)) %>%
  dplyr::mutate(Sentiment = ifelse(Sentiment == 'suspense', 'surprise', Sentiment)) %>%
  dplyr::mutate(Sentiment = ifelse(Sentiment == 'joyfulreunion' | Sentiment == 'joy in baking', 'joy', Sentiment)) %>%
  dplyr::mutate(Sentiment = ifelse(Sentiment == 'solace' | Sentiment == 'isolation', 'solitude', Sentiment)) %>%
  dplyr::mutate(Sentiment = ifelse(Sentiment == 'sadness', 'sad', Sentiment)) %>%
  dplyr::mutate(Sentiment = ifelse(Sentiment == 'pride', 'proud', Sentiment)) %>%

```
# Filtering and creating charts
1. The relationship between time spent on social media and the composite mental health score
   
The first graph shows the relationship between time spent on social media and the composite mental health score. We first calculated the composite mental health score from the different columns and grouped it by time spent on social media
```
social_media <- social_media %>%
  mutate(Composite_mental_health_Score = rowMeans(select(., `Distraction 1-5`, `Restlessness 1-5`, `Insomnia 1-5`, `Distraction level 1-5`, `Stress 1-5`, `Comparison and stress 1-5`), na.rm = TRUE))

```
We then created a chart that shows the relationship between them

2. Chart that shows how the mental health score vary accross different social media platforms

In this section, we created a graph that shows the mental health score variation accross different social media platforms such as facebook, instagram and twitter
```
ggplot(mental_health_summary, aes(x = Type, y = Mean_Score, fill = Type)) +
  geom_bar(stat = "identity", position = position_dodge(), color = "black") +
  scale_fill_manual(values = colors) +
  geom_errorbar(aes(ymin = Mean_Score - SD_Score, ymax = Mean_Score + SD_Score), width = 0.2, position = position_dodge(0.9)) +
  theme_minimal() +
  labs(title = "Average Mental Health Score by Social Media Platform",
       x = "Social Media Platform",
       y = "Average Mental Health Score") +
  theme(axis.text.x = element_text(angle = 45, hjust = 1), legend.position = "none")

```
# Sentiment analysis
One of the things we did to show the relationship between social media and mental health is to create a sentiment analysis

```
ggplot(filtered_afinn_sentiment, aes(x = reorder(word, sentiment), y = sentiment, fill = word)) +
  geom_bar(stat = "identity") +
  coord_flip() +
  labs(title = "AFINN Sentiment Analysis",
       x = "Sentiment",
       y = "Score") +
  theme_minimal()

```
2. Word cloud
We created a word cloud to show the most common positive and negative words from the dataset.
```
wordcloud_data <- tokens %>%
  inner_join(get_sentiments("bing")) %>%
  count(word, sentiment, sort = TRUE) %>%
  acast(word ~ sentiment, value.var = "n", fill = 0)
comparison.cloud(wordcloud_data, colors = c("red4", "blue3"), max.words = 100)

```

# Shiny app link:
https://fevenf.shinyapps.io/SocialMediaImpactOnMentalHealth/

1. Save the files in RDS format to make it easier to load and run the code efficiently.

```
mental_health_summary <- readRDS("Variability in Mental Health Scores Across Social Media Platforms.rds")
correlation_data <- readRDS("Correlation between Mental Health and Age & Gender.rds")
mental_health_by_time_bin <- readRDS("Social Media Time vs. Mental Health Score.rds")
wordcloud_data <- readRDS("Sentiment Wordcloud.rds")
bing_sentiment <- readRDS("Sentiment Analysis using BING lexicon.rds")
nrc_sentiment <- readRDS("Sentiment Analysis using NRC lexicon.rds")
afinn_sentiment <- readRDS("Sentiment Score Analysis.rds")

```
2. Define the UI
```
ui <- fluidPage(
  # App title
  titlePanel("Social Media and its Impact on Mental Health"),
  # Sidebar layout
  sidebarLayout(
    sidebarPanel(
      # Sidebar menu with choices
      selectInput("choice", "Choose an option:",
                  choices = c("Introduction",
                              "Sentiment Score Analysis",
                              "Sentiment Analysis using BING lexicon",
                              "Sentiment Analysis using NRC lexicon",
                              "Sentiment Wordcloud",
                              "Social Media Vs. Mental Health Score",
                              "Mental Health Summary",
                              "Correlation between Mental Health and Age & Gender"),
                  selected = "Discussion Article 1")
    ),
   
```
3. Define the server
```
server <- function(input, output) {
  # Function to render different outputs based on user choice
  output$output_content <- renderUI({
    switch(input$choice,
           "Introduction" = textOutput("Introduction"),
           "Sentiment Score Analysis" = plotOutput("afinn_sentiment"),
           "Sentiment Analysis using BING lexicon" = plotOutput("bing_sentiment"),
           "Sentiment Analysis using NRC lexicon" = plotOutput("nrc_sentiment"),
           "Sentiment Wordcloud" = plotOutput("wordcloud_data"),
           "Social Media Vs. Mental Health Score" = plotOutput("mental_health_by_time_bin"),
           "Mental Health Summary" = plotOutput("mental_health_summary_plot"),
           "Correlation between Mental Health and Age & Gender" = plotOutput("correlation_data"))
  })

# Run the application
shinyApp(ui = ui, server = server)
```
