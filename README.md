# askai — terminal AI for files, folders, and ZIPs

A small Windows-friendly command that sends a prompt plus readable project context to either the OpenAI API or Gemini API and streams the answer back into your terminal.

## What it supports

- A single source/text file: Python, C/C++, JS/TS/TSX, JSON, Markdown, CSV, YAML, SQL, and other common text/code formats.
- A folder: recursively scans supported files.
- A ZIP archive: reads supported files directly from the archive without extracting it.
- PDF, DOCX, PPTX, XLSX, and XLSM: extracts text/tables locally before sending context.
- Streams the answer to the terminal and can optionally save it to a file.

For PDFs and office documents, this starter extracts text/tables, not embedded images or visual diagrams. It is not a shell-execution agent and does not modify your source files.

## 1. Install

1. Install Python 3.11 or newer and make sure `python` works in PowerShell.
2. Extract this folder somewhere permanent, for example `C:\Tools\askai`.
3. Open PowerShell in that folder and run:

   ```powershell
   python -m pip install -r requirements.txt
   ```

4. Add the folder containing `askai.cmd` to your user PATH. One way is: Windows Settings → System → About → Advanced system settings → Environment Variables → select your user `Path` → Edit → New → add `C:\Tools\askai`. Open a new terminal after changing PATH.

## 2. Set API keys

You need a provider API key. A ChatGPT or Gemini app subscription does not automatically pay for API calls; API usage may be billed separately.

Create a private config file at `%USERPROFILE%\.askai\.env`:

```powershell
New-Item -ItemType Directory -Force "$HOME\.askai" | Out-Null
notepad "$HOME\.askai\.env"
```

Add one or both lines, replacing the placeholders with your own API keys:

```dotenv
OPENAI_API_KEY=your_openai_api_key_here
GEMINI_API_KEY=your_gemini_api_key_here
```

Save and close Notepad. Never put this file inside a project/repository or share it. API keys can incur charges.

Optional model overrides can be added to the same file:

```dotenv
OPENAI_MODEL=gpt-6-luna
GEMINI_MODEL=gemini-3.6-flash
ASKAI_PROVIDER=gemini
```

The defaults are examples and can be changed with `--model` if your API account supports another model.

## 3. Use it

```powershell
# Ask about the current folder (will prompt for provider and prompt)
askai

# Ask about the current project using Gemini
askai . --provider gemini

# Inspect a specific file using OpenAI
askai "src\main.py" --provider openai --prompt "Explain this file from first principles."

# Review a ZIP without manually extracting it
askai "C:\Users\you\Downloads\project.zip" --provider gemini --prompt "Review the architecture, identify bugs, and give fixes in priority order."

# Analyze a whole folder and save the answer
askai . --provider openai --prompt "Find likely bugs. Do not edit files; explain each fix." --save review.md

# Override the model
askai . --provider gemini --model gemini-2.5-pro --prompt "Explain the project."
```

If `--prompt` is omitted, the program asks for your prompt in the terminal. If `--provider` is omitted, it asks you to choose a provider unless `ASKAI_PROVIDER` is set.

## Safety and limits

- The program skips `.env` files, common credentials/private-key files, and generated folders such as `.git`, `node_modules`, `.venv`, `dist`, and `build`.
- Review files before sending them. It is still possible for secrets or private data to appear inside ordinary source files. Anything included in the context is sent to the selected provider's API.
- Only readable text and extracted document text is included; images, executables, and other unsupported binary files are skipped.
- Default limits: 120 files, 5 MB per file, 50 MB total scanned, and 180,000 context characters. If the context is too large, the script warns that some content may be omitted.
- API rate limits, model availability, and pricing depend on your provider and account.
