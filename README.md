[![Stand With Ukraine](https://raw.githubusercontent.com/vshymanskyy/StandWithUkraine/main/banner2-direct.svg)](https://stand-with-ukraine.pp.ua)

[![License GPL 3](https://img.shields.io/badge/license-GPL_3-green.svg)](http://www.gnu.org/copyleft/gpl.html)
[![MELPA](http://melpa.org/packages/pdb-capf-badge.svg)](http://melpa.org/#/pdb-capf)
# emacs-pdb-capf
`completion-at-point` function for `pdb` (Python debugger)

## Usage

Enable pdb completions for specific modes by adding `pdb-capf` to `completion-at-point-functions`, e.g. (for GUD):

```elisp
(add-hook 'pdb-mode-hook
          (lambda ()
            (add-hook 'completion-at-point-functions
                      'pdb-capf nil t)))
```

## Screenshot

<p align="center">
  <img src="https://raw.githubusercontent.com/muffinmad/emacs-pdb-capf/master/screenshots/datetime-completions.png" width="640">
</p>

# Improvement sugggestions from GPT:


[ME]: when completion-at-point called, pdb-capf--completions return the correct completion strings but completion-table-with-cache called two times and one of which includes unrelated global completions and in the end, there is no completion is found. It seems like when completion-table-with-cache first time, it has the related completions but then overriden by other completions... why?


[AI]: Without seeing the specific code in action it's hard to know for sure, but I can provide a few potential explanations and solutions:

1. `completion-table-with-cache` is designed to cache completion results for performance. But as the state of a pdb session may change minutes to minutes, so the cache might be causing troubles. It's possible that it's caching earlier completions that are no longer relevant.

    If possible, you can work around this by using `completion-table-dynamic` wrap around `pdb-capf--completions` instead of `completion-table-with-cache`. This makes the completions to be re-calculated every time, bypassing the cache:

    ```elisp
    (completion-table-dynamic #'pdb-capf--completions)
    ```
    
2. If switching to `completion-table-dynamic` doesn't resolve the issue, it's also possible that there's a race condition between the two calls to `completion-table-with-cache`. If the pdb process hasn't finished computing the completions by the time the second call is made, the second call might overwrite the result of the first call. 

   If this is the case, you can try to add a sleep or a loop that waits until the pdb process has generated the completions before calling `completion-table-with-cache` for the second time.

It's also worth noting that in your current implementation, `pdb-capf--completions` is fetching the completions synchronously: it waits until the completions have been returned before it finishes. This means that if the pdb process takes a while to generate the completions, Emacs will hang. You might want to refactor `pdb-capf--completions` to fetch the completions asynchronously to improve the user experience.
