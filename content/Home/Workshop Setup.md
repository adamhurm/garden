## Approaches to editing workshop notes
- Obsidian
	- Open Obsidian Vault via Terminal on macOS. We could use built-in [obsidian cli](https://obsidian.md/help/cli), but these commands don't require Obsidian.app to be running.
	  > **First time:**
		    ```
		   open "obsidian://open?path=/absolute/path/to/garden/content"```

	  > **Re-open:**
		    ```
		   open "obsidian://vault/content"```
- [Neovim](https://neovim.io/doc/install/) with [lazy.nvim](https://lazy.folke.io/installation) plugins:
	- [render-markdown](https://github.com/MeanderingProgrammer/render-markdown.nvim) (runs entirely inside nvim)
	- [markdown-preview](https://github.com/iamcco/markdown-preview.nvim) (requires web browser)
