# Character Creation Process Documentation

This documentation is written to explain how the prompting should work with examples.

## Breakdown of each line

### "max_context_length": 4096
`max_context_length` is used to specify to the language model how many tokens maximum it is going to process. When we start the model we specify if it will use 512, 1024, 2048, 4096 tokens etc. This number should be the amount set during startup. By default, we set it to 4096.

### "max_length": 512
`max_length` is to describe how many tokens we allow the model to return back. Test numbers between 512 and 1024.

### "rep_pen": 1.15
`rep_pen` is a basic repetition penalty. Higher values make the output less repetitive. If the character is fixated on something or repeats the same phrase, then increasing this parameter can help fix it. It is not recommended to increase this parameter too much as it may break the outputs. To disable it use value 1.

### "temperature": 0.99
`temperature` is a setting that controls the randomness of the generated text. The lower values provide more logical but less creative answers while higher values create more creative but less logical answers.

Before the next 3 values, I need you to understand the following concepts. We will be talking about top p sampling, top k sampling, and top a sampling. This is how it works simplified. LLM to pick the next word gets around 32 thousand different choices per word and it iterates over them using values assigned. top p and top k sampling are most commonly used and best ways to generate text but top a has its own niche which you can mess around with.

For this explanation, I am going to create a very simplified example and explain it.

As an llm, we get these 4 choices for the next word and its weighted (probability) value:

- Word A: 0.4
- Word B: 0.3
- Word C: 0.2
- Word D: 0.1

### "top_p"
Top p is called nucleus sampling. We set a number between 0 and 1 as a decimal, and it will use as the probability of the next word. If we set a top p of 0.9, it will calculate the highest probability tokens until the probability adds up to 90%. Thus, the higher the number, the more words it has to choose from, leading to more creative but less consistent responses and with lower value, the text is more consistent but less creative.

For example, if we set top p to 0.7, it will choose to consider the smallest set of words that combined have a total value of at least 0.7. So with our example, we add the biggest to smallest we will choose word A (0.4) and word B (0.3). That will give us a value of 0.7, so our choice will be selected from those two values.

But here is the catch. If we set a lower number of 0.3 as our top p, it will look for values that are at or above 0.3 from highest to lowest. In turn, it will add word A (0.4) and stop as it exceeded the threshold of 0.3, so it will only consider that singular word.

### "top_k"
Top K is called the most likely. This sampling limits the pool of choices from the total number of choices down to the number of highest numbered words we want to consider. We set a number between 1 and no top limit. This means if we chose a lower number, our pool gets smaller, so our responses get more predictable. The higher the number, the more words we pass, giving more rarer words a chance.

For example, if we set our top K to 2 for the above example, we will only get a selection of word A and word B as they have the highest value.

### "top_a"
Top A is our adaptive sampling. It has a more adaptive and flexible approach. In top A, we don't threshold combined probability like top p nor the number of words like top k. With top a, we select probability values themselves. Meaning we set a threshold, and any word above the threshold is considered. The method is dynamically adjusted, so the more higher probability words we get, the bigger our resource pool. The number is from 0-1 with.

For example, if we set our threshold of top a as 0.15 on our previous example, it will give us a pool of word A because 0.4 > 0.15, word B because 0.3 > 0.15, and word C because 0.2 > 0.15. It will not choose word D because 0.1 < 0.15.

Knowing that the recommended values for best consistency and performance are top p of 0.9, top k of 20-50, and top a of 0.

To disable them set:
- top p as 1
- top k as 0
- top a as 0

### Recommended values
"top_p": 0.9
"top_k": 50
"top_a": 0

Next typical sampling.

### "typical": 1
Typical sampling chooses a word randomly from the list, and all the words get the same chance of being selected, so it's recommended to disable this by setting it to 1.

### "tfs": 0.5
`tfs` is tail-free sampling. This setting removes the least probable words from consideration during the generation process, which can improve the quality and coherence. To disable, set to 1.

### "rep_pen_range": 320
`rep_pen_range` is how many tokens the model will look back over within your conversation to check for repeating phrases.

### "rep_pen_slope": 0.7
`rep_pen_slope` is a coefficient that determines the severity of the penalty to repeating tokens. Before choosing if the model finds the token repeating, it will multiply the number to lower the chance of it being selected. The lower the number, the fewer repetitions encouraging variety. The higher the number, the less the penalty.

### "sampler_order": [6, 0, 1, 3, 4, 2, 5]
`sampler_order` shows which order to run the sampler to pick probabilities. Make sure not to change this, as it can cause really bad results and possibly break the llm.

Before getting to memory, I'm going to go through the rest of them.

### "min_p": 0.4
`min_p` is an experimental value as of right now that is the opposite of top p, and the threshold you chose will remove the bottom least probable tokens not to exceed the threshold. Setting it to 0 disables it.

### "dynatemp_range": 0
`dynatemp_range` is a dynamic temperature setting where we give it two numbers, and it will set the temperature between them automatically. Recommended to have it at 0 to disable it as it is unstable.

### "presence_penalty": 0.5
Presence penalty is another penalty for words that have already existed. Recommended to keep it at 0 or close to 0 to minimize penalizing the llm multiple times for the same choice.

### "logit_bias": {}
`logit_bias` is encouraging the llm to use certain words and prioritize the chance of specific tokens appearing in the output. An advanced concept that is not needed for use cases and should be left at default.

### "prompt": ""
Prompt is where the previous conversation history goes into, and the new conversation goes into.

### "quiet": true
Setting quiet within the prompt and `--quiet` within the launcher gives players more privacy as their requests will not appear on the terminal.

### "stop_sequence": []
Stop sequence is used when creating the character to figure out when to stop the generation of new text. If a stop sequence is triggered, it means the llm will stop. In case AI starts responding for the user, adjust the stop sequence with where it needs to stop the generation.

### "use_default_badwordsids": false
Leave this as false, so the llm sends back an end-of-stream token so we know it is done generating.

### "streaming": true
Set streaming to true to stream the tokens from the llm instead of sending back one big chunk.

### "appendText": "\nTyrone:"
AppendText is the way we indicate it's the character's time to speak. Make sure to put \n then the character's name and :. Make sure that there are no spaces after the :

### "voiceID": "z9fAnlkpzviPz146aGWa"
Voice ID is used to set the voice used from ElevenLabs.

### "stability": "0.47"
Stability is the stability of the voice to send to ElevenLabs as well.

## Back to Memory

Memory has a lot of steps, but I'll go over them:
```plaintext
[
Character("Name of your character");
Species("Species name");
Age("xx" or "xx years old");
Physical appearance("Hair color, Eye color, Tattoos/scars/etc, xx cm, xx feet, xx inches tall, xx frame");
Clothes: (x top + x bottom);
Mind("trait1"+"trait2"+"trait3"+"trait4");
Personality("trait1"+"trait2"+"trait3"+"trait4");
Loves("x1"+"x2"+"x3"+"x4");
Hates("x1"+"x2"+"x3"+"x4");
Description("CharName is x" + "CharName enjoys x" + "CharName wants x" + "CharName uses x");
]
```

The `+` symbols are used to explain different things that can be put in it. When writing, please stick to writing with commas.

That should cover the entirety of the character creation process. Please take a look at a few example characters created that use features that you would want.
