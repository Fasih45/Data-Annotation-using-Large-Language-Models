# Data-Annotation-using-Large-Language-Models
Implementation of Google Gemini model to annotate twitter data.
In this guide, we’ll walk through the process of utilizing Large Language Models (LLMs) for data annotation and how to integrate them with annotation tools like Prodigy and BULK for efficient and high-quality dataset creation.

**1. Introduction**

Large Language Models (LLMs) have gained significant attention for their capability to understand and generate human-like text. Leveraging LLMs for data annotation can streamline the process, especially when dealing with large datasets. In this guide, we’ll demonstrate how to use LLMs to annotate text data, specifically focusing on tweet sentiment analysis as an example.
**
2. Setting Up the Environment**

Before we begin, ensure you have the necessary libraries installed. We’ll be using the google.generativeai library to interact with the generative AI model. Additionally, make sure you have access to an API key for the generative AI service.

![image](https://github.com/Fasih45/Data-Annotation-using-Large-Language-Models/assets/116684462/15e090d6-769a-4cc8-8c3b-a7069049d77c)

**3. Defining the Annotation Task**

For this guide, let’s consider the task of annotating tweet sentiments. We’ll categorize tweets as either Positive, Extremely positive, Negative, or Extremely Negative. Additionally, we’ll ask annotators to provide a brief explanation for their annotation choice.

**4. Generating Annotation Prompts**

To utilize the LLM effectively, we’ll provide it with prompts that guide the annotation process. We’ll define a set of prompts for annotating tweet sentiments.

prompts = [
    "Annotate the following tweet: '{tweet}'. Only use one of the following labels: Positive, Extremely positive, Negative, Extremely Negative. Also Provide an explanation for the annotation.",
    "Analyze the tweet: '{tweet}'. Select a label from the following options to annotate the tweet: Positive, Extremely positive, Negative, Extremely Negative. Give a reason for the choice of selected label.",
    "Evaluate the sentiment expressed in the tweet: '{tweet}'. Pick one of the sentiment labels: Positive, Extremely positive, Negative, Extremely Negative. Justify your selection with a brief explanation."
]

**5. Cleaning Text Data**

Before feeding tweets to the LLM, it’s essential to clean the text data. We’ll define functions to clean both tweet text and generated responses.
![image](https://github.com/Fasih45/Data-Annotation-using-Large-Language-Models/assets/116684462/f2645a82-f68f-4c6e-b68d-09d0d47073af)

**6. Annotating Data**

Now, we’ll read tweets from a CSV file and generate annotations using the LLM based on the defined prompts. Annotations will include the sentiment label and an explanation.

with open("Corona_NLP_test.csv", "r", encoding="utf-8") as csvfile:
    reader = csv.DictReader(csvfile)
    fieldnames = ["No.", "tweet", "Prompt", "generated annotations", "explanation"]

    with open("corona_NLP_test_annotated.csv", "w", newline="", encoding="utf-8") as outfile:
        writer = csv.DictWriter(outfile, fieldnames=fieldnames)
        writer.writeheader()

        for i, row in enumerate(reader, start=1):
            tweet = row["OriginalTweet"]
            # Clean tweet text
            tweet = clean_tweet(tweet)

            # Randomly select a prompt
            prompt = random.choice(prompts).format(tweet=tweet)

            try:
                convo = model.start_chat(history=[])
                convo.send_message(prompt)

                if convo and convo.last:
                    response = convo.last.text

                    # Check if the response contains safety settings message
                    if "HARM_CATEGORY" in response:
                        label = "Safety setting triggered"
                        reason = response.strip()
                    else:
                        # Check if the response contains annotations and reasons
                        if "." in response:
                            # Split response into annotations and reasons
                            parts = response.split(".")
                            if len(parts) > 1:  # Ensure both annotation and reason are present
                                # The first part contains the annotation
                                label = parts[0].strip()

                                # The second part contains the reason
                                reason = ".".join(parts[1:]).strip()
                                reason = clean_response(reason)  # Clean response text

                                allowed_labels = {"Positive", "Extremely positive", "Negative", "Extremely Negative"}
                                if label not in allowed_labels:
                                    label = "Unknown sentiment"
                            else:
                                label = "Neutral"
                                reason = "No specific reason given"
                        else:
                            label = "Neutral"
                            reason = "No specific reason given"

                else:
                    label = "Error: Unable to get response from the model"
                    reason = "Error: Unable to get response from the model"
            except Exception as e:
                label = "Error: Unable to analyze tweet"
                reason = "Error: Unable to analyze tweet"

            writer.writerow({
                "No.": i,
                "tweet": tweet,
                "Prompt": prompt,
                "generated annotations": label,
                "explanation": reason
            })

Results

![image](https://github.com/Fasih45/Data-Annotation-using-Large-Language-Models/assets/116684462/0466d007-3037-4a50-a2cc-78f126472b83)

