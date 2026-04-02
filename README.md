# Hey, YOU! 

## yes, YOU! 

do YOU want to ask nick land what he thinks the societal implications of tung tung tung sahur are? WELL NOW YOU CAN!

this... is the LANDLING. a very very very (very...) small language model that i finetuned to generate text in the style of the sacred texts (aka nick land's x account @xenocosmography). the landling is a TinyLlama-1.1B model finetuned using LoRA on a manually curated dataset of Land's tweets and replies

when i say manually curated, i am NOT kidding. i literally sat there and went through his page collecting around 200ish solo tweets and reply pairs formatted as prompt/completion pairs. 200 is laughably small for a finetuning dataset but i wasn't able to get a scraper working so you gotta make do with what you got. mama didn't raise a quitter. the lemurs didn't either.

fair warning: this model is very small and very bad. it will occasionally say something great and then immediately follow it up with something nonsensical

i will update it as frequently as a very busy overachieving high school senior can. i understand this is really shitty — i am kinda sorta on a time crunch and did not gather as much data as i should have. however, i really enjoyed working on this and will try to make 1-2-3 updates a month depending on how much time i have. so, this will be less shitty. lots of love lol

## example outputs

> God is
> 
> "God is about to judge the world a lot worse than he did last time."

> Why do you post so much about Jews?
> 
> "Because it's a conspiracy theory that has no empirical support."

> What happens after the singularity?
> 
> "Silly you."

> Do you trust the plan?
> 
>"Sure, but I don't believe in it."

> Retardation is
> 
> "Retardation is now the norm."

## technical stuff 

- base model: TinyLlama/TinyLlama-1.1B-Chat-v1.0
- finetuning: LoRA via HuggingFace PEFT + TRL
- epochs: 6
- hardware: Google Colab T4 GPU (free tier, i am an unemployed high schooler)

## model

hosted on hf: [clarezahermetica/landling](https://huggingface.co/clarezahermetica/landling)
