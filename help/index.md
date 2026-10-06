# AI prompts and tagging

Run your own prompts on texts and images with large language models (LLMs), or let a model tag
your files. The same prompt can run on different models and providers, so you can compare them.

## How to run a prompt

1. Pick a text or an image (or a set of them) and open **Crunchers**.
2. Choose **AI prompts**.
3. Choose a **prompt**. Text prompts are offered for texts, image prompts for images.
4. Choose a **model**. Only models that can read this kind of file are shown.
5. Choose a **provider** if the model is available from several. The badges tell where your file
   goes: *on-premise* stays on our servers, *external* is sent to the provider's cloud.
6. Optionally set the temperature and the maximum answer length, then run.

The answer appears as a new file next to the input: `<file name>.txt`, or `.json` for prompts
with JSON output.

## Writing prompts

Prompts are kept under **Prompts** in the menu. Each prompt has:

- **Type**: *Text to text* (the file's text is given to the model) or *Image to text* (the image
  is given to the model).
- **Prompt**: the instructions, e.g. *"Summarise this letter in three sentences. Name the sender
  and the recipient."* The file itself is added automatically; do not paste it into the prompt.
- **Output**: plain text, or **JSON**. For JSON give a schema, either a real JSON Schema or just
  an example of the answer you want:

  ```json
  { "sender": "", "recipient": "", "year": 1900, "places": [""] }
  ```

  The model must then answer with exactly these fields, so the results of many files can be
  compared or turned into a table.

Public prompts are shared by everyone; your own prompts are visible only to you.

## Tag with AI

**Tag with AI** chooses tags for each file:

- **Categories given** (typed as a comma list, or existing tags picked from your tags): the model
  may only use those. Tag descriptions help it decide. Check *Allow several tags* if a file can
  have more than one.
- **No categories**: the model suggests up to *Max suggested tags* short subject tags.
- **Extra guidance** is added to the instructions, e.g. *"tag the main topic only"*.

The tags are linked to the file as machine-made tags, so you can filter and search by them. Works
on texts and, with models that read images, on images too.

## Limits

- **Text length.** Each model takes a limited amount of text. A text that is too long fails with
  a message telling how long it was; split it first (for example by pages).
- **Tokens.** Commercial providers bill per token (about 3/4 of a word). Your service group may
  limit how many tokens you and the group can use per month. When the limit is reached, new runs
  are refused and a running set is paused; you can resume it when the limit has been raised or
  the month has turned.
- **Results vary.** Models can be wrong, invent facts and answer differently on each run. Check
  the results before relying on them; a low temperature makes answers more repeatable.

## Privacy

On-premise providers (Ollama, vLLM) run on our own servers and nothing leaves them. External
providers (OpenAI, Azure OpenAI, Google Gemini) receive the file you run the prompt on. Do not
send personal or confidential material to external providers unless your data management plan
allows it. Access to external providers is given per service group.
