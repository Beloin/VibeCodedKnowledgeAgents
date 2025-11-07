# Orchestrator.Agent - Knowledge Repository Builder

This project contains the Orchestrator.Agent - a specialized AI agent that coordinates the creation of comprehensive knowledge repositories through a sequential workflow.

## 🎯 How to Use Orchestrator.Agent

### Basic Usage
Simply tell the Orchestrator.Agent what subject you want to search for:

```
Search [subject]
```

**Examples:**
- "Search Computer Networks"
- "Search Rust Programming"
- "Search OpenCode"

### What Happens Next

The Orchestrator.Agent will automatically:

1. **Create Directory Structure**
   - Creates `repo_[subject]` folder
   - Organizes all files systematically

2. **Call Searcher.Agent**
   - Researches the subject comprehensively
   - Creates `key_concepts.v1.md` with foundational knowledge

3. **Call Validator.Agent**
   - Validates accuracy and truthfulness
   - Creates validated versions if needed
   - Iterates until content is accurate

4. **Call KnowledgeCrunch.Agent**
   - Expands each key concept into detailed guides
   - Creates comprehensive expanded documentation

5. **Organize Resources**
   - Moves all files to `resources/` subfolder
   - Creates final `[subject]_key_concepts.md`

6. **Create README with Roadmap**
   - Generates entry point README
   - Provides structured learning path
   - Links all available resources

## 🛠️ Available Agents

- **Orchestrator.Agent** - You are here! Coordinates the entire workflow
- **Searcher.Agent** - Researches and creates initial content
- **Validator.Agent** - Validates accuracy and truthfulness
- **KnowledgeCrunch.Agent** - Expands concepts into detailed guides

## 📚 Example Workflow

```
User: Search Computer Networks

Orchestrator.Agent:
1. Creates repo_computer_networks/
2. Calls Searcher.Agent → creates key_concepts.v1.md
3. Calls Validator.Agent → validates content
4. Calls KnowledgeCrunch.Agent → creates expanded guides
5. Organizes files into resources/
6. Creates README.md with learning roadmap
```

## 🎓 Result

You get a complete, structured knowledge repository with:
- Key concepts overview
- Detailed expanded guides
- Learning roadmap
- Organized file structure
- Validated, accurate content

---

*Ready to create a knowledge repository? Just say "Search [your subject]"*
