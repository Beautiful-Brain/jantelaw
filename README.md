# Demo
Explore the data in [jantelaw.replit.app](https://jantelaw.replit.app)

# Hypothesis
**Can LLM’s be used for research?**
*how LLMs can help us speed up research by quickly summarizing the perception (at least on the Internet) of something complicated as culture.*
- LLMs are a compression of the entire Internet, so they should be a good representation of the average opinion of humans - note there will be a bias as the average online human is not the same as the average human, but assuming it is close
- LLM’s provide a flexibility of LLM’s that allow us to generate text in a very flexible way, for example “write from a perspective of a doctor” or “write what the average Cuban thinks about X”
- With LLM’s it also came advancements in embeddings algorithms, that allow to transform pieces of text into numbers that represent the semantic meaning, which allow transform
- Given that I recently learned about Law of Jante, a set of 10 rules that are set to represent Danish culture, I thought it could be a good exercise to use that and try to compare how similarity culture wise are the different cultures in the World

# Design
![Design Diagram](https://github.com/zemigsan/jantelaw/blob/main/diagram.png?raw=true)

# Generating Law of Jante (Prompt Engineering)
**Generating the Law of Jante for each country**

- First I had to get a list of countries. I decided to go with the United Nations list of countries, which can be seen [here](https://www.un.org/en/about-us/member-states).
- I tried two prompting approaches: 0-shot (where I just ask to generate a Law of Jante for country) and 1-shot (where I give the example of Law of Jante laws for Denmark and ask to generate similar to country x)
- I just did some simple tests for countries that I know well (Portugal and USA) and decided to go with the 1-shot prompt (prompt below)
- Then I used colab to generate all the laws of jante for all countries, total cost was around $4, which turns out to be about $0.02 per country

![Costs of creating the Prompts with GPT 4](https://github.com/zemigsan/jantelaw/blob/main/gpt4costs.png?raw=true)

**Prompt I am using to generate for each country**
```
Generate a list of ten cultural rules in the style of the Jante Law for the country {country}.

Use the same style as this example for Denmark:
1. Humility and Modesty: You're not to think you are anything special.
2. Equality and Uniformity: You're not to think you are as good as we are.
3. Collective Wisdom over Individual Knowledge: You're not to think you are smarter than we are.
4. Egalitarianism: You're not to imagine yourself better than we are.
5. Respect for Collective Experience: You're not to think you know more than we do.
6. Community Focus over Individual Importance: You're not to think you are more important than we are.
7. Self-effacement and Anti-Exceptionalism: You're not to think you are good at anything.
8. Solidarity and Respect for Others: You're not to laugh at us.
9. Self-reliance within the Community: You're not to think anyone cares about you.
10. Collective Learning and Shared Knowledge: You're not to think you can teach us anything.
```


# Matching rules for comparison (Embeddings)
**Finding a way to compare countries rules**
- Now that I had the list of rules for each country, I had to find a way to compare two countries' list of rules
- To do that, I first got the OpenAI embeddings (text-embedding-ada-002) to convert each rule into semantic vector
- After that, I used the Hungarian algorithm with cosine similarity, to have for each country pair, the pairs of rules that are more close semantically with each other, and their cosine similarity score
- At first I thought I would just use the average of the cosine similarities of pair of rules to calculate the country similarity score, but I found it was ineffective because
  - the range was very small, with the cosine similarity of rules just flowing between 0.8 and 1.0
  - two pair of rules that are opposite of each other (eg. one focus on individuality and other focused on collective wisdom) would have higher cosine similarity than two very unrelated rules
- so the embeddings were helpful to match the more similar pairs of rules, but were not enough to understand which rules are cultural similar with each other
- embeddings cost were very cheap, costing less than $0.03 to embed the 2000 rules




# Matching rules for comparison (Prompt Engineering)
- To solve the cultural similarity categorization of each rule pair, I decided to use GPT3.5 (cheaper and it is easier task)
- I created a prompt that asked to provide a rationale on how similar are the rules (chain of thought prompting), and to give me a score of -5 to +5
- I had some difficulties making sure the model adhere to provide the score, with the first attempt only giving me 29.5% of the time
- After adding "PLEASE MAKE SURE YOU ALWAYS OUTPUT THE RATIONALE AND THE NUMERIC SCORE" to the bottom, that dropped to 23.4%
- One other issue was the amount of time that this required, as there was almost 200k pair of rules, which was taking more than a day to run at my first attemp. I did this things to speed up the process:
  - I decided to automatically classify the rule pairs as 5, when the cosine similarity was higher than 0.95 (I tested some examples and they were basically the same rule) - this also saved money
  - I used asyncio to call OpenAI in paralell, sending 100 requests at any given time. This improved speed a lot, and put the process down to a couple of hours. One issue is that the results were failing at 9% rate, but I made sure I was saving the progress and could run it again just to fill the ones that failed.
- in the end the costs were 

**Prompt**
```
Please rate these two cultural rules in terms of how similar they are with a score of -5 to +5
(-5 means they are the opposite of each other, 0 means they are unrelated/neutral to each other,
and +5 means they are basically the same)

Rule 1: {rule1} 
Rule 2: {rule2}

Please output in the following format:
Rationale: explain the reasoning around scoring
Score: (value between -5 and 5)
PLEASE MAKE SURE YOU ALWAYS OUTPUT THE RATIONALE AND THE NUMERIC SCORE
```


| Group               | Percentage Missing GPT-3 Score |
|--------------------|---------------------------------|
| Prompt without CAPS | 29.5%                          |
| Prompt with CAPS    | 23.4%                          |


# Similarity Score Algorithm

![Costs of creating the Prompts with GPT 4](https://github.com/zemigsan/jantelaw/blob/main/distributionscores.png?raw=true)

