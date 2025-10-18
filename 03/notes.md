# Signal Forms 

**[Stackblitz Interactive Preview](stackblitz.com/edit/signal-forms)**
<br>
Will still be experimental in v21... :(

Goal - take the best parts of TemplateForms, ReactiveForms, and Signals 

## Blurry Pictures 
![Example signal form](resources/01_signal-form-control.jpg)
*Example of a signal form's fields*

![Types of each property on the form](resources/02_sf-types.jpg)
*Types of each property accessible from the form*

# AI Stuff 

[Guide to NG with AI](https://angular.dev/ai/develop-with-ai)
- Includes standardized starting prompts to guide the AI in what it procudes

**Did you know?!**: This way of doing for loops is deprecated
![AI trained on old code](resources/03_ai-gens-old-code.jpg)
*AI trained on old code generates things that are out of date*

@for is how modern loops should be written according to the ng folks
![AI with context](resources/04_ai-gen-new-code.jpg)
*AI with additional context about what makes good angular code*

You can achieve this by writing good prompts (see the NG AI link)
![Good prompting](resources/07_good-prompt-guidelines.jpg)

Be careful about ballooning context window sizes!
![Large context windows](resources/05_ai-prompting-buckets.jpg)

Large context windows are big contributors to what causes AI to generate bad code (or bad information in general). As the window gets larger, the context window loses more and more of the beginning part of the conversation and prioritizes more recent information, causing it to lose important context and instructions.
![As the context window grows, the usefulness shrinks](resources/06_buckets-grow-quick.jpg)