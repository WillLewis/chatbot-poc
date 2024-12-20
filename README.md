# RAG-based Knowledge Bot POC Notes

## Summary
- **Goal:** Create a Proof-of-Concept (POC) of a Knowledge Bot for client using Retrieval-Augmented Generation (RAG) within 3 days.
- **Approach:** 
  1. Scrape all relevant client content from a dynamic React.js knowledge base web app for client.
  2. Convert scraped data into PDFs for observability.
  3. Use embeddings from OpenAI to power a retrieval-augmented chatbot.
  4. Evaluate acceptance rate, which ended up at **98%** from internal stakeholders.

## Implementation Details

### Created Data Pipeline (the hardest part)
1. **Objective:** Scrape data from the web app into PDFs.
   
2. **Browser Automation:**
   1. **Initial Attempt:**
      - Started with Selenium for automated browser tasks.
      - Found Selenium less effective on highly dynamic JS-rendered content (~70% success).
   2. **Improvement:**
      - Switched to Puppeteer for better handling of dynamic, JS-loaded sites.
      - Achieved ~98% data scraping success.

3. **Authorization Handling:**
   - Integrated login flows with Puppeteer.
   - Managed sessions/tokens to ensure continuous access.

4. **UI Content Extraction:**
   - Waited for specific CSS selectors to ensure content is fully loaded before extraction.
     - This step increased captured content by ~50%.
   - Manipulated the UI (e.g., expanding accordions, clicking "Show more" buttons) to reveal hidden text.
   - Implemented a recursive link-following mechanism:
     - Extracted linked PDFs and PPT files.
     - Skipped or filtered out video content.

5. **Data Parsing and Formatting:**
   - Extracted text content from the loaded DOM.
   - Applied CSS optimizations for clean, uniform formatting.
   - Saved extracted content into PDF files to maintain a verifiable data audit trail.

### Data Science Steps
1. **Data Storage:**
   - Stored raw text and extracted content in a Redis blob storage before embedding.
   - Ensured indexing and metadata tagging to facilitate efficient retrieval.

2. **Embeddings Creation:**
   - Used OpenAI’s embedding model (e.g., `text-embedding-ada-002`) to generate vector embeddings from the parsed content.
   - Considered chunking longer documents into smaller passages (e.g., 500-1000 tokens per chunk) for more accurate retrieval.

3. **Vector Database:**
   - Used Redis vector indexes to optimize retrieval latency.

4. **Model Integration:**
   - Built a multi-turn knowledge bot using  OpenAI API as the language model and Hugging Face as front-end.
   - RAG approach:
     - On query, retrieve top-k relevant embeddings from vector storage.
     - Construct a prompt with retrieved context.
     - Send prompt to the LLM to generate the answer.

5. **Fine-Tuning & Hyperparameters:**
   - Fine-tuned the base language model parameters for better alignment with Brand Central’s domain.
   - Optimal fine-tuning values for use case:
     - **Temperature:** 0.7 (balance creativity vs. factual accuracy)
     - **Frequency Penalty:** 0.5 (reduce repetition of frequently used terms)
     - **Presence Penalty:** 0.0 (allow model to introduce new topics if relevant)
     - **Stop:** `["\n\n"]` (stop token to prevent model from rambling)
     - **n:** 1 (single best completion for simplicity)
     - **top_p:** 1.0 (no sampling restriction, relying on temperature)

6. **Prompt Engineering:**
   - Created purpose-built prompts:
     - Included system messages with role and domain context.
     - Used a few-shot prompting approach, providing examples of question-answer pairs based on scraped content.
   - Employed chain-of-thought prompting to encourage step-by-step reasoning where possible.

7. **Evaluation Strategy:**
   - Conducted internal acceptance testing:
     - Compared the bot’s answers to known ground-truth documents.
     - Checked for accuracy, completeness, and adherence to brand guidelines.
   - Measured precision/recall on a sample set of Q&A pairs derived from the knowledge base.
   - Collected qualitative feedback from fellow engineers and brand managers.
   - Achieved a 98% acceptance rate based on test queries, indicating successful POC validation.

### Additional Steps for Production
1. **Add Data Quality Checks:**
   - Create programmatic system for matching extracted text to source.
   - Ensured that non-textual content (images, charts) are handled gracefully (omitted or described if text available).

2. **Latency Optimization:**
   - Pre-cached embeddings of frequently queried content.
   - Used lightweight retrieval methods (e.g., approximate nearest neighbor search) for faster response times.

3. **Continuous Improvement:**
   - Planned integration of automated retraining when the knowledge base updates.
   - Considered using OpenAI Function Calling or improved RAG frameworks for more structured responses in future iterations.

---
