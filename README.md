# Customer Service Robot

This is a  Customer Service Robot that utilizes Openai API key to fulfill its function. It is a chatbot but different from a traditional one, which can only answer prefixed questions. However, this chatbot is also different than chatgpt which can only provide you the general answer as it doesn't have specific data. Therefore, this customer service  chatbot combines both strengths since it can provide you the AI-generated answer for unusual questions  and standard answer for frequent-asked questions.

Here is the basic illustration of Chatbot:

![csvfile](images\161650.png)

prerequisite:

- Node.js installed
- python installed
- openai apikey
- a csv or json file that presentsprefixed question and answer

The first thing we need to do is to make a fine-tuned model by using the library of openai:

1. First, write a Python script that divides into two parts- prompts and completion. Feed each segment to Chatgpt and ask him to ask relevant questions based on the text content. Record the questions and answers in the file wingit.csv. The format is as follows:

   ![csvfile](images\210038.png)

2. Then format the data in the JSONL format required for the fine-tune model:

   ~~~
   openai tools fine tunes.prepare data -f wingit.csv
   ~~~

   The result is as follows:

   ![Jsonlfile](images\214105.png)

3. Set the key for open AI. Set your environment variables by adding the following line to your shell initialization script (such as. bashrc. zshrc, etc.) or running it from the command line before the fine-tuning command: (The following are all run from Window command line)

   ~~~
   $export OPENAI API KEY="XXX or $env:OPENAI_API_KEY="xxx"
   ~~~

4. Create a fine-tuning job:

   ~~~
   openai api fine_tunes.create -t wingit_prepared.jsonl -m davinci
   ~~~

5. Sometimes, as this is a brand-new practice, your training will be interrupted. At this time, you need to continue the training.

   ~~~
   openai api fine_tunes.follow -i <YOUR FINE TUNE JOB ID>
   ~~~

After the training we can get our unique fine-tune model, which looks like follows:

~~~
openai api fine_tunes.get -i ft-K9oA99Xcivw9u2uc5MavOjje
[2023-08-29 15:51:11] Created fine-tune: ft-ZvX8Fw6hK2EbaIyn70DDSr3P
[2023-08-29 15:51:16] Fine-tune costs $0.26
[2023-08-29 15:51:16] Fine-tune enqueued. Queue number: 0
[2023-08-29 16:01:18] Fine-tune started
[2023-08-29 16:03:12] Completed epoch 1/4
[2023-08-29 16:03:25] Completed epoch 2/4
[2023-08-29 16:03:37] Completed epoch 3/4
[2023-08-29 16:03:49] Completed epoch 4/4
[2023-08-29 16:04:23] Uploaded model: davinci:ft-personal-2023-08-29-06-04-23
[2023-08-29 16:04:24] Uploaded result file: file-mKt1F753oAjYi5g41XxFrfrm
[2023-08-29 16:04:24] Fine-tune succeeded

Job complete! Status: succeeded 🎉
Try out your fine-tuned model:

openai api completions.create -m davinci:ft-personal-2023-08-29-06-04-23 -p <YOUR_PROMPT>
~~~

**davinci:ft-personal-2023-08-29-06-04-23** is our personal fine-tune model. Copy it into Index.js file

~~~
async function fetchReply(){
    const response = await openai.createCompletion({
        model: 'davinci:ft-personal-2023-08-23-01-36-15', //changed from 'davinci'
        prompt: conversationStr, //changed from 'messages'
        presence_penalty: 0,
        frequency_penalty: 0.3,
        max_tokens: 200,
        temperature: 0,
        stop: ['\n', '->'], //GPT4 has a stop sequence built in so we dont need to add it for that, but we do need it for our custom data. We also add the '->' to keep arrows from popping up in completions, and to tell the AI that it has reached the end of a prompt.
    })
    console.log("API Response:", response);
    // we need to explicitly add a white space to the beginning to fulfill the prompt-completion pair requirement
    // we also needed to add an '\n' to the end to make sure the AI knows that it has reached the end of a completion.
    conversationStr +=  response.data.choices[0].text  // we change '.push()' to '+=', 'message' to 'text' because that's what we get back from the API, we also remove the () around response.
    renderTypewriterText(response.data.choices[0].text)
}
~~~

Finally, run this chatbot in the terminal

```
npm install
npm start
````
