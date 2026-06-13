# FitFindr — planning.md

> Complete this document before writing any implementation code.
> Your spec and agent diagram are what you'll use to direct AI tools (Claude, Copilot, etc.) to generate your implementation — the more specific they are, the more useful the generated code will be.
> Your planning.md will be reviewed as part of your submission.
> Update it before starting any stretch features.

---

## Tools

List every tool your agent will use. For each tool, fill in all four fields.
You must have at least 3 tools. The three required tools are listed — add any additional tools below them.

### Tool 1: search_listings

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
It searches the listings json file for the clothes user has asked, and if it exists in the listings, the output of this tool is given to suggest_outfit tool, otherwise it returns nothing. 

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `description` (str): the outfit user wants is described here
- `size` (str): what size does the user want like ('S', 'M', 'L', 'XL')
- `max_price` (float): The maximum price that user wants their clothes to be, they will not accept any clothes above this value.

**What it returns:**
<!-- Describe the return value — what fields does a result contain? -->
Returns top k matching listings sorted by relevance. 
Fields - "Fit that user asked", price of that fit and the condition it's in.   

**What happens if it fails or returns nothing:**
<!-- What should the agent do if no listings match? -->
If tool 1 doesn't return anything the user asks for, then agent suggests user to try something different and stops, it will not go for the next tool

---

### Tool 2: suggest_outfit

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
It takes the output of tool 1, along with the wardrobe schema file, to identify how the listings outfit would match with existing wardrobe, and if it does, it gives user what outfits they should wear with the listing one, with some styling advice.  

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `new_item` (dict): This is a dictionary that has the fit user has asked, the price of it and also what coniditon it is in, basically the output of tool 1
- `wardrobe` (dict): This one is a dictionary from the wardrobe schema file, which includes all of the items (id, name, category, colors, style_tag, notes) for best match with listings outfit and to give the user best outfit match. 

**What it returns:**
<!-- Describe the return value -->
It returns a string that describes which wardrobe outfit to pair with the one listings has returned, while giving styling advice. 

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the wardrobe is empty or no outfit can be suggested? -->
If nothing can be suggested, instead of going to the next tool, the agent should say there's no good suggestion or that wardrobe is empty, and tell the user appropriate message depending on the case - either to add their wardrobe or suggesting them to find a different outfit. 

---

### Tool 3: create_fit_card

**What it does:**
<!-- Describe what this tool does in 1–2 sentences -->
It looks at the suggested outfit and the outfit it took from listings to give a caption that will talk about how well the thrift fit fits the user while also mentioning any good qualities about it - be it either the price or the quality. 

**Input parameters:**
<!-- List each parameter, its type, and what it represents -->
- `outfit` (str): This input is taken from tool 2, and it has the required information for the agent to print out a caption from it
- `new_item` (dict): This is the output from too 1, containing information regarding the thrift fit, it's price and the condition it's in. 

**What it returns:**
<!-- Describe the return value -->
Using above input parameters, it would return a string that will complement thrift fit with the user while talking about any good qualities about it. 

**What happens if it fails or returns nothing:**
<!-- What should the agent do if the outfit data is incomplete? -->
Agent should tell the user that it doesn't have enough information about the outfit to suggest a good caption, and tell user to either add more wardrobe data or try searching for a different fit in the thrift. 

---

### Additional Tools (if any)

<!-- Copy the block above for any tools beyond the required three -->

---

## Planning Loop

**How does your agent decide which tool to call next?**
<!-- Describe the logic your planning loop uses. What does it look at? What conditions change its behavior? How does it know when it's done? -->
Check user query, load the wardrobe from the schema/session at the start. 
**Step 1**: Call search_listings with inputs taken from the query. Once search_listing runs, if the result is empty or is giving an error, then set an error message in this session and return to the user saying, "Sorry, I couldn't find any listings matching your query. You can try a different description or raise your budget."

If no, set selected_item = results[0] then proceed to call suggest_outfit.

**Step 2**: Now, call suggest_outfit with inputs taken from tool 1 and user wardrobe. Once it runs, check if the result is empty or if no outfit can be suggested, and if true, set an error message in this session and return to the user saying, "I found a listing but couldn't pair it with your wardrobe, try searching for another fit or add more items to your wardrobe." If no, proceed to calling create_fit_card. 

**Step 3**: Check the result of create_fit_card and if it is empty or if it's incomplete, agent should put an error message for this session and return back to user saying, "I don't have enough information to create a fit card, you can add more to your wardrobe or change your inital thrift fit, but here's the fit information you asked." If it succeeds, agent now creates a string message to the user saying they found the fit user is looking for and tells how to match it with their existing wardrobe while also giving a nice caption that it produced from tool 3. With this final string, it knows it's job is done. 

---

## State Management

**How does information from one tool get passed to the next?**
<!-- Describe how your agent stores and accesses state within a session. What data is tracked? How is it passed between tool calls? -->

---

## Error Handling

For each tool, describe the specific failure mode you're handling and what the agent does in response.

| Tool | Failure mode | Agent response |
|------|-------------|----------------|
| search_listings | No results match the query | |
| suggest_outfit | Wardrobe is empty | |
| create_fit_card | Outfit input is missing or incomplete | |

---

## Architecture

<!-- Draw a diagram of your agent showing how the components connect:
     User input → Planning Loop → Tools (search_listings, suggest_outfit, create_fit_card)
                                                                          ↕
                                                                   State / Session
     Show what triggers each tool, how state flows between them, and where error paths branch off.
     ASCII art, a Mermaid diagram (https://mermaid.js.org/syntax/flowchart.html), or an embedded
     sketch are all fine. You'll share this diagram with an AI tool when asking it to implement
     the planning loop and each individual tool. -->

---

## AI Tool Plan

<!-- For each part of the implementation below, describe:
     - Which AI tool you plan to use (Claude, Copilot, ChatGPT, etc.)
     - What you'll give it as input (which sections of this planning.md, your agent diagram)
     - What you expect it to produce
     - How you'll verify the output matches your spec before moving on

     "I'll use AI to help me code" is not a plan.
     "I'll give Claude my Tool 1 spec (inputs, return value, failure mode) and ask it to implement
     search_listings() using load_listings() from the data loader — then test it against 3 queries
     before trusting it" is a plan. -->

**Milestone 3 — Individual tool implementations:**

**Milestone 4 — Planning loop and state management:**

---

## A Complete Interaction (Step by Step)

Write out what a full user interaction looks like from start to finish — tool call by tool call. Use a specific example query.

**Example user query:** "I'm looking for a vintage graphic tee under $30. I mostly wear baggy jeans and chunky sneakers. What's out there and how would I style it?"

**Step 1:**
<!-- What does the agent do first? Which tool is called? With what input? -->
First, it calls the search_listings tool with search_listings("vintage graphic tee", size = "N/A", max_price = 30.00)

**Step 2:**
<!-- What happens next? What was returned from step 1? What tool is called now? -->
Next, it takes the top result from search_listings and calls suggest_outfit tool with suggest_outfit(new_item = 'listing from step 1', wardrobe = 'user wardrobe containing baggy jeans and chunky sneakers'). It suggests the new listing outfit with existing wardrobe items, while also giving advice on how to style it. It returns a suggested outfit and styling advice.

**Step 3:**
<!-- Continue until the full interaction is complete -->
Once step 2 is done, it will call creat_fit_card with create_fit_card(outfit = 'outfit from step 2', new_item = 'listing from step 1') to create a visual fit card for the user. It returns something like an instagram caption describing the outfit and how well it fits the user. 

**Final output to user:**
<!-- What does the user actually see at the end? -->
They would see the agent suggesting them a vintage graphic tee under $30, telling them how to style it with their baggy jeans and chunky sneakers, along with a fit card showing how the outfit looks together and a caption describing the fit and style advice.
