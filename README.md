# iOS-AI-Style-Guide

This repo contains two guides that I include in my system prompts when using LLMs in my iOS project. This ensures the generated code matches my personal style and preferences.

## How To Use

This depends on how you are using an LLM, but generally you want to include something like the following in your system/project prompt (this is what I use for [AlexSideBar](https://www.alexcodes.app/)):

"Use the guides in the alexsidebar directory at the root of the project directory. Use the project-guide.md file in that directory to understand my preferred file structure for an Xcode project when creating new files, which will help you determine where they should be placed. Update the file structure if it does not match my preferences. Use the style-guide.md in the same directory to understand how Swift code should be written and how different types relate to each other. Follow the conventions and architecture in the style guide for the specified types. Do not create your own architecture. Both files are contained in the alexsidebar directory."

### Claude Code

When using Claude Code, you will probably want to distill these two guides into shorter list items that can be added to your CLAUDE.md file. You can even have Claude do that for you!

## Contribution

Feel free to open PRs to improve the guides with reccomendations! I am just starting to dive into using AI as a tool, so any helpful pointers would be appreciated.