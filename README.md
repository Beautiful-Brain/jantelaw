# Demo
Explore the data in [jantelaw.replit.app](https://jantelaw.replit.app)

# Hypothesis
**Can LLM’s be used for research?**
*how LLMs can help us speed up research by quickly summarizing the perception (at least on the Internet) of something complicated as culture.*
- LLMs are a compression of the entire Internet, so they should be a good representation of the average opinion of humans - note there will be a bias as the average online human is not the same as the average human, but assuming it is close
- LLM’s provide a flexibility of LLM’s that allow us to generate text in a very flexible way, for example “write from a perspective of a doctor” or “write what the average Cuban thinks about X”
- With LLM’s it also came advancements in embeddings algorithms, that allow to transform pieces of text into numbers that represent the semantic meaning, which allow transform
- Given that I recently learned about Law of Jante, a set of 10 rules that are set to represent Danish culture, I thought it could be a good exercise to use that and try to compare how similarity culture wise are the different cultures in the World

# Prompt Engineering
**Generating the Law of Jante for each country**

- First I had to get a list of countries. I decided to go with the United Nations list of countries, which can be seen here.
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


# Algorithm for comparison - Embeddings
**Generate algorithm**
-start by use embeddings and cosine similiray, hungarian algorithm
- using gpt 3 to fill the gaps - % numbers / cost
- scaling gpt3 


| group | percentage_missing_gpt3_score | percentage_missing_rationale |
|-------|-------------------------------|------------------------------|
| A/B   | 29.45%                        | 0.00%                        |
| Other | 18.11%                        | 9.16%                        |

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
