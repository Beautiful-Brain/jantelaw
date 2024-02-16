# Demo
Explore the data in [jantelaw.replit.app](https://jantelaw.replit.app)

# Hypothesis
**Can LLM’s be used for research?**
*how LLMs can help us speed up research by quickly summarizing the perception (at least on the Internet) of something complicated as culture.*
- LLMs are a compression of the entire Internet, so they should be a good representation of the average opinion of humans - note there will be a bias as the average online human is not the same as the average human, but assuming it is close
- LLM’s provide a flexibility of LLM’s that allow us to generate text in a very flexible way, for example “write from a perspective of a doctor” or “write what the average Cuban thinks about X”
- With LLM’s it also came advancements in embeddings algorithms, that allow to transform pieces of text into numbers that represent the semantic meaning, which allow transform
- Given that I recently learned about Law of Jante, a set of 10 rules that are set to represent Danish culture, I thought it could be a good exercise to use that and try to compare how similarity culture wise are the different cultures in the World

# Learnings
**Data**
- We are more similar than we think - the more common rules were shared by most countries: Humility and Modesty (84% of countries share), Respect for Tradition (79%), Collective Wisdom over Individual Knowledge (65%) and Respect for Elders (62%)
- The top 3 countries that were most different:
    - USA - interestingly the only country the model decided to put the rules in the positive vs negative, also it is the country where GPT has more information about so was able to get more specific
    - North Korea - obvious reasons
    - Norway - for some reason the model put Norwegians in most answers, which didn't do in all other countries (strong nationalism in Norway?)
- Most similar countries pairs tend to come from Africa - hypothesis is where GPT has less information about and tends to generalize

**Prompting**
- Prompt engineering is indeed a thing, for example, when I made a small change in a prompt I was able to reduce the % of missing values by 6%. Still lots of improvements I can make here with more time.
- Prompt engineering costs, input tokens can be significantly important, I was trying to save costs by only asking for the score and putting more context tokens, and end up spending 50% more
- Things change fast, I ended up spending about $100 on the total project, and just after it OpenAI decided to slash prices of GPT3 by 50%, which would have saved me much money, the takeaway is costs of AI are decreasing fast

# Design
![Design Diagram](https://github.com/zemigsan/jantelaw/blob/main/diagram.png?raw=true)

# Generating the Law of Jante (Prompt Engineering)
**Generating the Law of Jante for each country**

- First, I had to compile a list of countries. I opted for the United Nations list of member states, accessible [here](https://www.un.org/en/about-us/member-states).
- I experimented with two approaches to prompting: 0-shot (where I directly request the generation of a Law of Jante for a country) and 1-shot (where I provide an example of the Law of Jante for Denmark and then request the generation of a similar set for country x).
- I conducted preliminary tests for countries I am familiar with (Portugal and the USA) and decided to proceed with the 1-shot prompt (detailed below).
- Subsequently, I utilized Colab to generate the Laws of Jante for all countries, incurring a total cost of around $4, which equates to approximately $0.02 per country.


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
**Finding a way to compare countries' rules**
- Now that I had compiled the list of rules for each country, I needed to find a method to compare the lists of rules between pairs of countries.
- To accomplish this, I first utilized OpenAI embeddings (text-embedding-ada-002) to convert each rule into a semantic vector.
- Afterward, I applied the Hungarian algorithm with cosine similarity to identify, for each pair of countries, the pairs of rules that were most semantically similar to each other, along with their cosine similarity scores.
- Initially, I considered using the average of the cosine similarities of rule pairs to calculate the country similarity score, but I found this approach to be ineffective because:
  - The range was very narrow, with the cosine similarity of rules fluctuating between 0.8 and 1.0.
  - A pair of opposite rules (e.g., one focusing on individuality and the other on collective wisdom) could have a higher cosine similarity than two unrelated rules.
  - Thus, while the embeddings were useful in matching the most similar pairs of rules, they were insufficient for determining which rules were culturally similar to each other.
- The cost of embeddings was very low, totaling less than $0.03 for embedding the 2000 rules.


# Classifier of rule similarity (Prompt Engineering)
- To solve the cultural similarity categorization of each rule pair, I decided to use GPT3.5 (cheaper and it is easier task)
- I created a prompt that asked to provide a rationale on how similar the rules are (chain of thought prompting), and to give me a score of -5 to +5
- I had some difficulties ensuring the model adhered to provide the score, with the first attempt not giving me the score 29.5% of the time. After adjusting to adding "PLEASE MAKE SURE YOU ALWAYS OUTPUT THE RATIONALE AND THE NUMERIC SCORE" to the bottom, that dropped to 23.4%
- One other issue was the amount of time that this required, as there were almost 200k pairs of rules, which was taking more than a day to run on my first attempt. I did these things to speed up the process:
  - I decided to automatically classify the rule pairs as 5 when the cosine similarity was higher than 0.95 (I tested some examples, and they were basically the same rule) - this also saved money
  - I used asyncio to call OpenAI in parallel, sending 100 requests at any time. This improved speed a lot and slowed the process to a couple of hours. One issue is that the results were failing at a 9% rate, but I made sure I was saving the progress and could run it again just to fill the ones that failed.
- The costs of running it to the almost 200k pairs of rules were $40

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

# Second Classifier of rule similarity (Prompt Engineering)
- To address the missing values on the 23% and to try to improve the scores, I decided to run a different prompt with a few-shot technique
- To try to make it cheaper, and because I already had the rationale, I decided to give a couple of examples of rationales and scores
- The costs of running it to the almost 200k pairs of rules were $60, 50% more than what I expected, I misunderestimate the cost of prompt tokens, that are cheaper but still cost 1/3 of the output tokens, so in the end, it ended up costing more
- It had a 100% completion rate of the task though, I was able to fill all the missing values

**Prompt**
```
Here are examples of rationales, that describe how two cultural rules compare with each other and their respective score. Scores range from +5 (if they completely agree with each other) to -5 (if they are in complete disagreement with each other). A score of 0 is used for neutral scoring, where the rules are not related to each other at all. A positive score indicates a degree of agreement or similarity between the rules, with higher scores indicating greater similarity. A negative score indicates disagreement or contrast between the rules, with lower scores indicating greater disparity.

Example 1:
Rationale: Rule 1 emphasizes the importance of the community and discourages individualism, while Rule 2 promotes individualism and self-reliance. These two rules seem to be in direct opposition to each other, as one prioritizes the collective and the other emphasizes the individual.
Score: -5

Example 2:
Rationale: Both Rule 1 and Rule 2 emphasize the importance of respecting others and not imposing one's beliefs on others. However, Rule 1 specifically focuses on family values, while Rule 2 pertains to political views. While these two rules may have some overlap in terms of promoting respect and avoiding imposition, they are not directly related to each other.
Score: 0

Example 3:
Rationale: Both Rule 1 and Rule 2 emphasize the importance of respect, but they focus on different aspects. Rule 1 highlights the need for religious tolerance and discourages the belief that one's own faith is superior to others. On the other hand, Rule 2 emphasizes the importance of preserving and not exploiting natural resources. While they address different domains (religion and nature), they both promote the idea of respecting something beyond oneself.
Score: 3

Example 4:
Rationale: The two cultural rules are identical in their wording and meaning. Both emphasize the importance of respecting and not considering oneself wiser than elders. Since they are exactly the same, they should receive the highest similarity score.
Score: 5

Example 5:
Rationale: The two cultural rules seem to have conflicting messages. Rule 1 emphasizes humility and modesty, suggesting that individuals should not think they are superior to others. On the other hand, Rule 2 emphasizes the importance of appearance, suggesting that individuals should not neglect their appearance. While both rules address individual behavior and attitudes, they seem to have different focuses and priorities.
Score: -3

Now, please rate the following rationale in terms of similarity, considering the definitions of scoring:

Rationale: {rationale}

PLEASE JUST OUTPUT THE NUMERIC SCORE - JUST THE NUMBER PLEASE
```

# Country Similarity Score Algorithm
- My final challenge was to combine the scores of the different pairs of rules into a unified country similarity score across two countries
- My first approach was just to use the score from gpt3 (-5 to 5), and sum each of the ten rules with 10 (so 5 would be 10, and -5 would be 0), but this led into very high scores, as the rules that were contradicting were not having a lot of impact

 ```python
def normalize_gpt3_score(score):
    """Normalize the GPT-3 score from -5 to +5 to a scale of 0 to 1."""
    return (score + 5) / 10 if score is not None else None
```
![Similarity Score Distribution (original algorithm)](https://github.com/zemigsan/jantelaw/blob/main/distributionscores.png?raw=true)
- Also I realized after watching some points, that both the old gpt3 score and new gpt3 score had flaws and contradict each other in some occasions, and the old seemed to be better decision maker (the new one tend to classify something as 0 more frequently), so I end up adding the following algorithm described below:

```python
def normalize_gpt3_score_v2(gpt3_score,gpt3_score_new, similarity_score):
  score = gpt3_score_new
  if gpt3_score <0 and gpt3_score_new>=0:
    score = gpt3_score
  if gpt3_score == 0:
    score = 0
  if(score >0):
    score = (score + 5) * similarity_score / 10
  elif score < 0:
    score /=10
  return score
```

![Similarity Score Distribution (current algorithm)](https://github.com/zemigsan/jantelaw/blob/main/distributionscoresv2.png?raw=true)

