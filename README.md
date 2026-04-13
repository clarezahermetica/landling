# Hey, YOU! 

## yes, YOU! 

do YOU want to ask nick land what he thinks the societal implications of tung tung tung sahur are? WELL NOW YOU CAN!

this... is the LANDLING. a very very very (very...) small language model that i finetuned to generate text in the style of the sacred texts (aka nick land's x account @xenocosmography). the landling is a TinyLlama-1.1B model finetuned using LoRA on a manually curated dataset of Land's tweets and replies

when i say manually curated, i am NOT kidding. i literally sat there and went through his page collecting around 200ish solo tweets and reply pairs formatted as prompt/completion pairs. 200 is laughably small for a finetuning dataset but i wasn't able to get a scraper working so you gotta make do with what you got. mama didn't raise a quitter. the lemurs didn't either.

fair warning: this model is very small and very bad. it will occasionally say something great and then immediately follow it up with something nonsensical

i will update it as frequently as a very busy overachieving high school senior can. i understand this is really shitty — i am kinda sorta on a time crunch and did not gather as much data as i should have. however, i really enjoyed working on this and will try to make 1-2-3 updates a month depending on how much time i have. so, this will be less shitty

i also wrote a whole thing about this [here](https://open.substack.com/pub/243165/p/i-finetuned-nick-lands-x-account) — what the landling is, why i made it, everything that changed in v1 and why, and how to actually use him yourself.

## example outputs

> God is
> 
> "God is about to judge the world a lot worse than he did last time."

> What do you think about AI alignment?
> 
> "AI Alignment (AA) is a misnomer. It's not an objective goal, and it doesn't refer to any particular technology or social system that needs aligning with respect to human values -- rather, AA refers only to the moral character of those who pursue such goals as themselves."

> The singularity is
> 
> "The singularity is finally happening. You can either be part of it, or watch from the sidelines."

> Pancakes or Waffles?
>
> "Waffles. 'Cause you'll only put pancakes in the trash if it sucks, and this thing just sucked."

(i asked him this 15 times. he said waffles all 15 times. the man loves his waffles lol)

> Do you believe in God? 
> 
> "Of course, but I don't think he cares."

## technical stuff

- base model: TinyLlama/TinyLlama-1.1B-Chat-v1.0
- finetuning: LoRA via HuggingFace PEFT + TRL
- epochs: 7
- hardware: Google Colab T4 GPU (free tier, i am an unemployed high schooler)
- dataset: 298 examples total (90 solo tweets, 138 reply pairs, 70 threaded conversation turns)
- trainable parameters: 2,252,800 out of 1.1B (0.2%!!!! I freaking love LoRA)
- training loss: 1.21 (down from 2.59 in v1!!)
- training time: ~754 seconds (~12.5 minutes. yeah it went up lol. more data will do that)

## changelog

### v1 (current)
- **added a third data category**: threaded conversations (longco). instead of just solo tweets and simple reply pairs, the model now trains on full multi-turn threads formatted cumulatively. each of Land's replies in a thread becomes its own training example with all prior context included. more signal, better conversational coherence
- **fixed a critical silent bug**: the solo tweet CSV was being read with a wrong column assignment, causing 89 out of 90 tweets to be silently dropped. the model was training on literally 1 solo tweet this whole time. this is very embarrassing and also explains a lot
- **dataset grew from ~200 to 298 examples** as a result of both the bug fix and the new data category
- **training loss dropped from 2.59 → 1.21** (huge!!!!)
- **epochs tuned to 7**: found through trial and error that 6 underfit and 8+ caused the model to start speaking in a cursed Italian/Spanish/Latin hybrid that doesn't exist in any of those languages. 7 is the sweet spot
- **generation parameters tightened**: temperature 0.7 → 0.6, added top_p=0.9, repetition penalty 1.3 → 1.4
- **added interactive chat loop** for multi-turn conversations with the Landling

### v0
- initial release. he spoke! but he spoke in tongues. great, but could've been better

## model

hosted on hf: [clarezahermetica/landling](https://huggingface.co/clarezahermetica/landling)
