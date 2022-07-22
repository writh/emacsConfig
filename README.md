Writh’s GNU Emacs Config

TABLE OF CONTENTS

IMPORTANT! PUT THIS IN YOUR INIT.EL
ABOUT THIS CONFIG
A FEW PROGRAMS TO LOAD FIRST
Setup Package.el To Work With MELPA and ELPY
Use-Package
Evil Mode
General Keybindings
STARTUP PERFORMANCE
Garbage collection
Native Compile
ALL THE ICONS
BUFFERS AND BOOKMARKS
DASHBOARD
Configuring Dashboard
Dashboard in Emacsclient
DELETE SELECTION MODE
ELFEED
EMOJIS
EVALUATE ELISP EXPRESSIONS
FILE MANAGER (DIRED)
Keybindings To Open Dired
Keybindings Within Dired
Keybindings For Peep-Dired-Mode
FILES
File-related Keybindings
Installing Some Useful File-related Modules
Useful File Functions
FONTS
Setting The Font Face
GENERAL KEYBINDINGS
GRAPHICAL USER INTERFACE TWEAKS
Disable Menubar, Toolbars and Scrollbars
Display Line Numbers and Truncated Lines
Change Modeline To Doom’s Modeline
IVY (COUNSEL/SWIPER)
Installing Ivy And Basic Setup
Making M-x Great Again!
Ivy-posframe
LANGUAGE SUPPORT
MAGIT
NEOTREE
ORG MODE
Defining A Few Things
Enabling Org Bullets
Org Link Abbreviations
Org Todo Keywords
Source Code Block Tag Expansion
Source Code Block Syntax Highlighting
Automatically Create Table of Contents
Make M-RET Not Add Blank Lines
Org Export To Manpage Format
PERSPECTIVE
PROJECTILE
REGISTERS
SCROLLING
SHELLS
Eshell
Vterm
IDE SETUP
Python
SPLITS AND WINDOW CONTROLS
THEME
STRAIGHT.EL
Telegram - Telga
WHICH KEY
RUNTIME PERFORMANCE
WRITEROOM MODE
IMPORTANT! PUT THIS IN YOUR INIT.EL

I don’t want to use init.el to config Emacs. I want to use an org file to config Emacs because I like literate configs with lots of comments. The following code block should be your init.el. This tells init.el to use the source code blocks from this file (config.org).

(org-babel-load-file
 (expand-file-name
  "config.org"
  user-emacs-directory))
ABOUT THIS CONFIG

This is my personal Emacs config. I patterned my config to mimic Doom Emacs, which is a distribution of Emacs that uses the “evil” keybindings (Vim keybindings) and includes a number of nice extensions and a bit of configuration out of the box. I am maintaining this config not just for myself, but also for those that want to explore some of what is possible with Emacs. I will add a lot of examples of plugins and settings, some of them I may not even use personally. I do this because many people following me on YouTube and Odysee look at my configs as “documentation”.

A FEW PROGRAMS TO LOAD FIRST

The order in which the various Emacs modules load is very important. So the very first code block is going to contain essential modules that many other modules will depend on later in this config.

Setup Package.el To Work With MELPA and ELPY

(require 'package)
(add-to-list 'package-archives
             '("melpa" . "https://melpa.org/packages/")
             '("elpy" . "http://jorgenshaefer.github.io/packages/"))
(package-refresh-contents)
(package-initialize)
Use-Package

Install use-package and enable ‘:ensure t’ globally. The ‘:ensure’ keyword causes the package(s) within use-package statements to be installed automatically if not already present on your system. To avoid having to add ‘:ensure t’ to every use-package statement in this config, I set ‘use-package-always-ensure’.

(unless (package-installed-p 'use-package)
  (package-install 'use-package))
(setq use-package-always-ensure t)
Evil Mode

Evil is an extensible ‘vi’ layer for Emacs. It emulates the main features of Vim, and provides facilities for writing custom extensions. Evil Collection is also installed since it adds ‘evil’ bindings to parts of Emacs that the standard Evil package does not cover, such as: calenda, help-mode adn ibuffer.

(use-package evil
  :init      ;; tweak evil's configuration before loading it
  (setq evil-want-integration t) ;; This is optional since it's already set to t by default.
  (setq evil-want-keybinding nil)
  (setq evil-vsplit-window-right t)
  (setq evil-split-window-below t)
  (evil-mode))
(use-package evil-collection
  :after evil
  :config
  (setq evil-collection-mode-list '(dashboard dired ibuffer))
  (evil-collection-init))
(use-package evil-tutor)
General Keybindings

General.el allows us to set keybindings. As a longtime Doom Emacs user, I have grown accustomed to using SPC as the prefix key. General makes setting keybindings (especially with SPC) much easier. All of the keybindings we set later in the config depend on general being loaded.

(use-package general
  :config
  (general-evil-setup t))
STARTUP PERFORMANCE

This section is where it make emacs faster to load.

Garbage collection

Makes startup faster by reducing the frequency of garbage collection

;; Using garbage magic hack.
 (use-package gcmh
   :config
   (gcmh-mode 1))
;; Setting garbage collection threshold
(setq gc-cons-threshold 402653184
      gc-cons-percentage 0.6)

;; Profile emacs startup
(add-hook 'emacs-startup-hook
          (lambda ()
            (message "*** Emacs loaded in %s with %d garbage collections."
                     (format "%.2f seconds"
                             (float-time
                              (time-subtract after-init-time before-init-time)))
                     gcs-done)))

;; Silence compiler warnings as they can be pretty disruptive (setq comp-async-report-warnings-errors nil)
Native Compile

;; Silence compiler warnings as they can be pretty disruptive
(if (boundp 'comp-deferred-compilation)
    (setq comp-deferred-compilation nil)
    (setq native-comp-deferred-compilation nil))
;; In noninteractive sessions, prioritize non-byte-compiled source files to
;; prevent the use of stale byte-code. Otherwise, it saves us a little IO time
;; to skip the mtime checks on every *.elc file.
(setq load-prefer-newer noninteractive)
ALL THE ICONS

This is an icon set that can be used with dashboard, dired, ibuffer and other Emacs programs.

(use-package all-the-icons)
BUFFERS AND BOOKMARKS

(nvmap :prefix "SPC"
       "b b"   '(ibuffer :which-key "Ibuffer")
       "b c"   '(clone-indirect-buffer-other-window :which-key "Clone indirect buffer other window")
       "b k"   '(kill-current-buffer :which-key "Kill current buffer")
       "b n"   '(next-buffer :which-key "Next buffer")
       "b p"   '(previous-buffer :which-key "Previous buffer")
       "b B"   '(ibuffer-list-buffers :which-key "Ibuffer list buffers")
       "b K"   '(kill-buffer :which-key "Kill buffer"))
DASHBOARD

Emacs Dashboard is an extensible startup screen showing you recent files, bookmarks, agenda items and an Emacs banner.

Configuring Dashboard

(use-package dashboard
  :init      ;; tweak dashboard config before loading it
  (setq dashboard-set-heading-icons t)
  (setq dashboard-set-file-icons t)
  (setq dashboard-banner-logo-title "Emacs Is More Than A Text Editor!")
  ;;(setq dashboard-startup-banner 'logo) ;; use standard emacs logo as banner
  (setq dashboard-startup-banner "~/.emacs.d/emacs-dash.png")  ;; use custom image as banner
  (setq dashboard-center-content nil) ;; set to 't' for centered content
  (setq dashboard-items '((recents . 5)
                          (agenda . 5 )
                          (bookmarks . 3)
                          (projects . 3)
                          (registers . 3)))
  :config
  (dashboard-setup-startup-hook)
  (dashboard-modify-heading-icons '((recents . "file-text")
			      (bookmarks . "book"))))
Dashboard in Emacsclient

This setting ensures that emacsclient always opens on dashboard rather than scratch.

(setq initial-buffer-choice (lambda () (get-buffer "*dashboard*")))
DELETE SELECTION MODE

By default in Emacs, we don’t have ability to select text, and then start typing and our new text replaces the selection. Let’s fix that!

(delete-selection-mode t)
ELFEED

An RSS newsfeed reader for Emacs.

(use-package elfeed
  :config
  (setq elfeed-search-feed-face ":foreground #fff :weight bold"
        elfeed-feeds (quote
                       (("https://www.reddit.com/r/linux.rss" reddit linux)
                        ("https://www.reddit.com/r/commandline.rss" reddit commandline)
                        ("https://www.reddit.com/r/distrotube.rss" reddit distrotube)
                        ("https://www.reddit.com/r/emacs.rss" reddit emacs)
                        ("https://www.gamingonlinux.com/article_rss.php" gaming linux)
                        ("https://hackaday.com/blog/feed/" hackaday linux)
                        ("https://opensource.com/feed" opensource linux)
                        ("https://linux.softpedia.com/backend.xml" softpedia linux)
                        ("https://itsfoss.com/feed/" itsfoss linux)
                        ("https://www.zdnet.com/topic/linux/rss.xml" zdnet linux)
                        ("https://www.phoronix.com/rss.php" phoronix linux)
                        ("http://feeds.feedburner.com/d0od" omgubuntu linux)
                        ("https://www.computerworld.com/index.rss" computerworld linux)
                        ("https://www.networkworld.com/category/linux/index.rss" networkworld linux)
                        ("https://www.techrepublic.com/rssfeeds/topic/open-source/" techrepublic linux)
                        ("https://betanews.com/feed" betanews linux)
                        ("http://lxer.com/module/newswire/headlines.rss" lxer linux)
                        ("https://distrowatch.com/news/dwd.xml" distrowatch linux)))))

(use-package elfeed-goodies
  :init
  (elfeed-goodies/setup)
  :config
  (setq elfeed-goodies/entry-pane-size 0.5))

(add-hook 'elfeed-show-mode-hook 'visual-line-mode)
(evil-define-key 'normal elfeed-show-mode-map
  (kbd "J") 'elfeed-goodies/split-show-next
  (kbd "K") 'elfeed-goodies/split-show-prev)
(evil-define-key 'normal elfeed-search-mode-map
  (kbd "J") 'elfeed-goodies/split-show-next
  (kbd "K") 'elfeed-goodies/split-show-prev)
EMOJIS

Emojify is an Emacs extension to display emojis. It can display github style emojis like 😄 or plain ascii ones like :).

(use-package emojify
  :hook (after-init . global-emojify-mode))
EVALUATE ELISP EXPRESSIONS

I choose to use the format ‘SPC e’ plus ‘key’ for these (I also use ‘SPC e’ for ‘eww’ keybindings).

COMMAND	DESCRIPTION	KEYBINDING
eval-buffer	Evaluate elisp in buffer	SPC e b
eval-defun	Evaluate the defun containing or after point	SPC e d
eval-expression	Evaluate an elisp expression	SPC e e
eval-last-sexp	Evaluate elisp expression before point	SPC e l
eval-region	Evaluate elisp in region	SPC e r
(nvmap :states '(normal visual) :keymaps 'override :prefix "SPC"
       "e b"   '(eval-buffer :which-key "Eval elisp in buffer")
       "e d"   '(eval-defun :which-key "Eval defun")
       "e e"   '(eval-expression :which-key "Eval elisp expression")
       "e l"   '(eval-last-sexp :which-key "Eval last sexression")
       "e r"   '(eval-region :which-key "Eval region"))
FILE MANAGER (DIRED)

Dired is the file manager within Emacs. Below, I setup keybindings for image previews (peep-dired). I’ve chosen the format of ‘SPC d’ plus ‘key’.

Keybindings To Open Dired

COMMAND	DESCRIPTION	KEYBINDING
dired	Open dired file manager	SPC d d
dired-jump	Jump to current directory in dired	SPC d j
Keybindings Within Dired

COMMAND	DESCRIPTION	KEYBINDING
dired-view-file	View file in dired	SPC d v
dired-up-directory	Go up in directory tree	h
dired-find-file	Go down in directory tree (or open if file)	l
Keybindings For Peep-Dired-Mode

COMMAND	DESCRIPTION	KEYBINDING
peep-dired	Toggle previews within dired	SPC d p
peep-dired-next-file	Move to next file in peep-dired-mode	j
peep-dired-prev-file	Move to previous file in peep-dired-mode	k
(use-package all-the-icons-dired)
(use-package dired-open)
(use-package peep-dired)

(nvmap :states '(normal visual) :keymaps 'override :prefix "SPC"
               "d d" '(dired :which-key "Open dired")
               "d j" '(dired-jump :which-key "Dired jump to current")
               "d p" '(peep-dired :which-key "Peep-dired"))

(with-eval-after-load 'dired
  ;;(define-key dired-mode-map (kbd "M-p") 'peep-dired)
  (evil-define-key 'normal dired-mode-map (kbd "h") 'dired-up-directory)
  (evil-define-key 'normal dired-mode-map (kbd "l") 'dired-open-file) ; use dired-find-file instead if not using dired-open package
  (evil-define-key 'normal peep-dired-mode-map (kbd "j") 'peep-dired-next-file)
  (evil-define-key 'normal peep-dired-mode-map (kbd "k") 'peep-dired-prev-file))

(add-hook 'peep-dired-hook 'evil-normalize-keymaps)
;; Get file icons in dired
(add-hook 'dired-mode-hook 'all-the-icons-dired-mode)
;; With dired-open plugin, you can launch external programs for certain extensions
;; For example, I set all .png files to open in 'sxiv' and all .mp4 files to open in 'mpv'
(setq dired-open-extensions '(("gif" . "sxiv")
                              ("jpg" . "sxiv")
                              ("png" . "sxiv")
                              ("mkv" . "mpv")
                              ("mp4" . "mpv")))
FILES

File-related Keybindings

(nvmap :states '(normal visual) :keymaps 'override :prefix "SPC"
       "."     '(find-file :which-key "Find file")
       "f f"   '(find-file :which-key "Find file")
       "f r"   '(counsel-recentf :which-key "Recent files")
       "f s"   '(save-buffer :which-key "Save file")
       "f u"   '(sudo-edit-find-file :which-key "Sudo find file")
       "f y"   '(dt/show-and-copy-buffer-path :which-key "Yank file path")
       "f C"   '(copy-file :which-key "Copy file")
       "f D"   '(delete-file :which-key "Delete file")
       "f R"   '(rename-file :which-key "Rename file")
       "f S"   '(write-file :which-key "Save file as...")
       "f U"   '(sudo-edit :which-key "Sudo edit file"))
Installing Some Useful File-related Modules

Though ‘recentf’ is one way to find recent files although I prefer using ‘counsel-recentf’.

(use-package recentf
  :config
  (recentf-mode))
(use-package sudo-edit) ;; Utilities for opening files with sudo
Useful File Functions

(defun dt/show-and-copy-buffer-path ()
  "Show and copy the full path to the current file in the minibuffer."
  (interactive)
  ;; list-buffers-directory is the variable set in dired buffers
  (let ((file-name (or (buffer-file-name) list-buffers-directory)))
    (if file-name
        (message (kill-new file-name))
      (error "Buffer not visiting a file"))))
(defun dt/show-buffer-path-name ()
  "Show the full path to the current file in the minibuffer."
  (interactive)
  (let ((file-name (buffer-file-name)))
    (if file-name
        (progn
          (message file-name)
          (kill-new file-name))
      (error "Buffer not visiting a file"))))
FONTS

Defining our fonts. Right now I’m using Source Code Pro (SauceCodePro) from the nerd-fonts repository. Installed from the AUR, it does NOT include all variations of the font (such as italics). You can download the italics Source Code Pro font from the nerd-fonts GitHub though.

Setting The Font Face

(set-face-attribute 'default nil
  :font "Source Code Pro"
  :height 110
  :weight 'medium)
(set-face-attribute 'variable-pitch nil
  :font "Ubuntu Nerd Font"
  :height 120
  :weight 'medium)
(set-face-attribute 'fixed-pitch nil
  :font "Source Code Pro"
  :height 110
  :weight 'medium)
;; Makes commented text and keywords italics.
;; This is working in emacsclient but not emacs.
;; Your font must have an italic face available.
(set-face-attribute 'font-lock-comment-face nil
  :slant 'italic)
(set-face-attribute 'font-lock-keyword-face nil
  :slant 'italic)

;; Uncomment the following line if line spacing needs adjusting.
(setq-default line-spacing 0.12)

;; Needed if using emacsclient. Otherwise, your fonts will be smaller than expected.
(add-to-list 'default-frame-alist '(font . "Source Code Pro-11"))
;; changes certain keywords to symbols, such as lamda!
(setq global-prettify-symbols-mode t)
You can use the bindings CTRL plus =/- for zooming in/out. You can also use CTRL plus the mouse wheel for zooming in/out.

;; zoom in/out like we do everywhere else.
(global-set-key (kbd "C-=") 'text-scale-increase)
(global-set-key (kbd "C--") 'text-scale-decrease)
(global-set-key (kbd "<C-wheel-up>") 'text-scale-increase)
(global-set-key (kbd "<C-wheel-down>") 'text-scale-decrease)
GENERAL KEYBINDINGS

General.el allows us to set keybindings. As a longtime Doom Emacs user, I have grown accustomed to using SPC as the prefix key. It certainly is easier on the hands than constantly using CTRL for a prefix.

(nvmap :keymaps 'override :prefix "SPC"
       "SPC"   '(counsel-M-x :which-key "M-x")
       "c c"   '(compile :which-key "Compile")
       "c C"   '(recompile :which-key "Recompile")
       "h r r" '((lambda () (interactive) (load-file "~/.emacs.d/init.el")) :which-key "Reload emacs config")
       "t t"   '(toggle-truncate-lines :which-key "Toggle truncate lines"))
(nvmap :keymaps 'override :prefix "SPC"
       "m *"   '(org-ctrl-c-star :which-key "Org-ctrl-c-star")
       "m +"   '(org-ctrl-c-minus :which-key "Org-ctrl-c-minus")
       "m ."   '(counsel-org-goto :which-key "Counsel org goto")
       "m e"   '(org-export-dispatch :which-key "Org export dispatch")
       "m f"   '(org-footnote-new :which-key "Org footnote new")
       "m h"   '(org-toggle-heading :which-key "Org toggle heading")
       "m i"   '(org-toggle-item :which-key "Org toggle item")
       "m n"   '(org-store-link :which-key "Org store link")
       "m o"   '(org-set-property :which-key "Org set property")
       "m t"   '(org-todo :which-key "Org todo")
       "m x"   '(org-toggle-checkbox :which-key "Org toggle checkbox")
       "m B"   '(org-babel-tangle :which-key "Org babel tangle")
       "m I"   '(org-toggle-inline-images :which-key "Org toggle inline imager")
       "m T"   '(org-todo-list :which-key "Org todo list")
       "o a"   '(org-agenda :which-key "Org agenda")
       )
GRAPHICAL USER INTERFACE TWEAKS

Let’s make GNU Emacs look a little better.

Disable Menubar, Toolbars and Scrollbars

(menu-bar-mode -1)
(tool-bar-mode -1)
(scroll-bar-mode -1)
Display Line Numbers and Truncated Lines

(global-display-line-numbers-mode 1)
(global-visual-line-mode t)
Change Modeline To Doom’s Modeline

(use-package doom-modeline)
(doom-modeline-mode 1)
IVY (COUNSEL/SWIPER)

Ivy, counsel and swiper are a generic completion mechanism for Emacs. Ivy-rich allows us to add descriptions alongside the commands in M-x.

Installing Ivy And Basic Setup

(use-package counsel
  :after ivy
  :config (counsel-mode))
(use-package ivy
  :defer 0.1
  :diminish
  :bind
  (("C-c C-r" . ivy-resume)
   ("C-x B" . ivy-switch-buffer-other-window))
  :custom
  (setq ivy-count-format "(%d/%d) ")
  (setq ivy-use-virtual-buffers t)
  (setq enable-recursive-minibuffers t)
  :config
  (ivy-mode))
(use-package ivy-rich
  :after ivy
  :custom
  (ivy-virtual-abbreviate 'full
   ivy-rich-switch-buffer-align-virtual-buffer t
   ivy-rich-path-style 'abbrev)
  :config
  (ivy-set-display-transformer 'ivy-switch-buffer
                               'ivy-rich-switch-buffer-transformer)
  (ivy-rich-mode 1)) ;; this gets us descriptions in M-x.
(use-package swiper
  :after ivy
  :bind (("C-s" . swiper)
         ("C-r" . swiper)))
Making M-x Great Again!

The following line removes the annoying ‘^’ in things like counsel-M-x and other ivy/counsel prompts. The default ‘^’ string means that if you type something immediately after this string only completion candidates that begin with what you typed are shown. Most of the time, I’m searching for a command without knowing what it begins with though.

(setq ivy-initial-inputs-alist nil)
Smex is a package the makes M-x remember our history. Now M-x will show our last used commands first.

(use-package smex)
(smex-initialize)
Ivy-posframe

Ivy-posframe is an ivy extension, which lets ivy use posframe to show its candidate menu. Some of the settings below involve:

ivy-posframe-display-functions-alist – sets the display position for specific programs
ivy-posframe-height-alist – sets the height of the list displayed for specific programs
Available functions (positions) for ‘ivy-posframe-display-functions-alist’

ivy-posframe-display-at-frame-center
ivy-posframe-display-at-window-center
ivy-posframe-display-at-frame-bottom-left
ivy-posframe-display-at-window-bottom-left
ivy-posframe-display-at-frame-bottom-window-center
ivy-posframe-display-at-point
ivy-posframe-display-at-frame-top-center
NOTE: If the setting for ‘ivy-posframe-display’ is set to ‘nil’ (false), anything that is set to ‘ivy-display-function-fallback’ will just default to their normal position in Doom Emacs (usually a bottom split). However, if this is set to ‘t’ (true), then the fallback position will be centered in the window.

(use-package ivy-posframe
  :init
  (setq ivy-posframe-display-functions-alist
    '((swiper                     . ivy-posframe-display-at-point)
      (complete-symbol            . ivy-posframe-display-at-point)
      (counsel-M-x                . ivy-display-function-fallback)
      (counsel-esh-history        . ivy-posframe-display-at-window-center)
      (counsel-describe-function  . ivy-display-function-fallback)
      (counsel-describe-variable  . ivy-display-function-fallback)
      (counsel-find-file          . ivy-display-function-fallback)
      (counsel-recentf            . ivy-display-function-fallback)
      (counsel-register           . ivy-posframe-display-at-frame-bottom-window-center)
      (dmenu                      . ivy-posframe-display-at-frame-top-center)
      (nil                        . ivy-posframe-display))
    ivy-posframe-height-alist
    '((swiper . 20)
      (dmenu . 20)
      (t . 10)))
  :config
  (ivy-posframe-mode 1)) ; 1 enables posframe-mode, 0 disables it.
LANGUAGE SUPPORT

Adding packages for programming langauges, so we can have nice things like syntax highlighting.

(use-package haskell-mode)
(use-package lua-mode)
(use-package markdown-mode)
MAGIT

A git client for Emacs. Often cited as a killer feature for Emacs.

(setq bare-git-dir (concat "--git-dir=" (expand-file-name "~/.dotfiles")))
(setq bare-work-tree (concat "--work-tree=" (expand-file-name "~")))
;; use maggit on git bare repos like dotfiles repos, don't forget to change `bare-git-dir' and `bare-work-tree' to your needs
(defun me/magit-status-bare ()
  "set --git-dir and --work-tree in `magit-git-global-arguments' to `bare-git-dir' and `bare-work-tree' and calls `magit-status'"
  (interactive)
  (require 'magit-git)
  (add-to-list 'magit-git-global-arguments bare-git-dir)
  (add-to-list 'magit-git-global-arguments bare-work-tree)
  (call-interactively 'magit-status))

;; if you use `me/magit-status-bare' you cant use `magit-status' on other other repos you have to unset `--git-dir' and `--work-tree'
;; use `me/magit-status' insted it unsets those before calling `magit-status'
(defun me/magit-status ()
  "removes --git-dir and --work-tree in `magit-git-global-arguments' and calls `magit-status'"
  (interactive)
  (require 'magit-git)
  (setq magit-git-global-arguments (remove bare-git-dir magit-git-global-arguments))
  (setq magit-git-global-arguments (remove bare-work-tree magit-git-global-arguments))
  (call-interactively 'magit-status))

(use-package magit)
NEOTREE

Neotree is a file tree viewer. When you open neotree, it jumps to the current file thanks to neo-smart-open. The neo-window-fixed-size setting makes the neotree width be adjustable. NeoTree provides following themes: classic, ascii, arrow, icons, and nerd. Theme can be configed by setting “two” themes for neo-theme: one for the GUI and one for the terminal. I like to use ‘SPC t’ for ‘toggle’ keybindings, so I have used ‘SPC t n’ for toggle-neotree.

COMMAND	DESCRIPTION	KEYBINDING
neotree-toggle	Toggle neotree	SPC t n
neotree- dir	Open directory in neotree	SPC d n
;; Function for setting a fixed width for neotree.
;; Defaults to 25 but I make it a bit longer (35) in the 'use-package neotree'.
(defcustom neo-window-width 25
  "*Specifies the width of the NeoTree window."
  :type 'integer
  :group 'neotree)

(use-package neotree
  :config
  (setq neo-smart-open t
        neo-window-width 30
        neo-theme (if (display-graphic-p) 'icons 'arrow)
        ;;neo-window-fixed-size nil
        inhibit-compacting-font-caches t
        projectile-switch-project-action 'neotree-projectile-action) 
        ;; truncate long file names in neotree
        (add-hook 'neo-after-create-hook
           #'(lambda (_)
               (with-current-buffer (get-buffer neo-buffer-name)
                 (setq truncate-lines t)
                 (setq word-wrap nil)
                 (make-local-variable 'auto-hscroll-mode)
                 (setq auto-hscroll-mode nil)))))

;; show hidden files
(setq-default neo-show-hidden-files t)

(nvmap :prefix "SPC"
       "t n"   '(neotree-toggle :which-key "Toggle neotree file viewer")
       "d n"   '(neotree-dir :which-key "Open directory in neotree"))
ORG MODE

Org Mode is THE killer feature within Emacs. But it does need some tweaking.

Defining A Few Things

(add-hook 'org-mode-hook 'org-indent-mode)
(setq org-directory "~/Org/"
      org-agenda-files '("~/Org/agenda.org")
      org-default-notes-file (expand-file-name "notes.org" org-directory)
      org-ellipsis " ▼ "
      org-log-done 'time
      org-journal-dir "~/Org/journal/"
      org-journal-date-format "%B %d, %Y (%A) "
      org-journal-file-format "%Y-%m-%d.org"
      org-hide-emphasis-markers t)
(setq org-src-preserve-indentation nil
      org-src-tab-acts-natively t
      org-edit-src-content-indentation 0)
Enabling Org Bullets

Org-bullets gives us attractive bullets rather than asterisks.

(use-package org-bullets)
(add-hook 'org-mode-hook (lambda () (org-bullets-mode 1)))
Org Link Abbreviations

This allows for the use of abbreviations that will get expanded out into a lengthy URL.

;; An example of how this works.
;; [[arch-wiki:Name_of_Page][Description]]
(setq org-link-abbrev-alist    ; This overwrites the default Doom org-link-abbrev-list
        '(("google" . "http://www.google.com/search?q=")
          ("arch-wiki" . "https://wiki.archlinux.org/index.php/")
          ("ddg" . "https://duckduckgo.com/?q=")
          ("wiki" . "https://en.wikipedia.org/wiki/")))
Org Todo Keywords

This lets us create the various TODO tags that we can use in Org.

(setq org-todo-keywords        ; This overwrites the default Doom org-todo-keywords
        '((sequence
           "TODO(t)"           ; A task that is ready to be tackled
           "BLOG(b)"           ; Blog writing assignments
           "GYM(g)"            ; Things to accomplish at the gym
           "PROJ(p)"           ; A project that contains other tasks
           "VIDEO(v)"          ; Video assignments
           "WAIT(w)"           ; Something is holding up this task
           "|"                 ; The pipe necessary to separate "active" states and "inactive" states
           "DONE(d)"           ; Task has been completed
           "CANCELLED(c)" )))  ; Task has been cancelled
Source Code Block Tag Expansion

Org-tempo is a package that allows for ‘<s’ followed by TAB to expand to a begin_src tag. Other expansions available include:

Typing the below + TAB	Expands to …
<a	’#+BEGIN_EXPORT ascii’ … ‘#+END_EXPORT
<c	’#+BEGIN_CENTER’ … ‘#+END_CENTER’
<C	’#+BEGIN_COMMENT’ … ‘#+END_COMMENT’
<e	’#+BEGIN_EXAMPLE’ … ‘#+END_EXAMPLE’
<E	’#+BEGIN_EXPORT’ … ‘#+END_EXPORT’
<h	’#+BEGIN_EXPORT html’ … ‘#+END_EXPORT’
<l	’#+BEGIN_EXPORT latex’ … ‘#+END_EXPORT’
<q	’#+BEGIN_QUOTE’ … ‘#+END_QUOTE’
<s	’#+BEGIN_SRC’ … ‘#+END_SRC’
<v	’#+BEGIN_VERSE’ … ‘#+END_VERSE’
(use-package org-tempo
  :ensure nil) ;; tell use-package not to try to install org-tempo since it's already there.
Source Code Block Syntax Highlighting

We want the same syntax highlighting in source blocks as in the native language files.

(setq org-src-fontify-natively t
    org-src-tab-acts-natively t
    org-confirm-babel-evaluate nil
    org-edit-src-content-indentation 0)
Automatically Create Table of Contents

Toc-org helps you to have an up-to-date table of contents in org files without exporting (useful useful for README files on GitHub). Use :TOC: to create the table.

(use-package toc-org
  :commands toc-org-enable
  :init (add-hook 'org-mode-hook 'toc-org-enable))
Make M-RET Not Add Blank Lines

(setq org-blank-before-new-entry (quote ((heading . nil)
                                         (plain-list-item . nil))))
Org Export To Manpage Format

(use-package ox-man
  :ensure nil)
PERSPECTIVE

The Perspective package provides multiple named workspaces (or “perspectives”) in Emacs, similar to multiple desktops in window managers like Awesome and XMonad. Each perspective has its own buffer list and its own window layout. This makes it easy to work on many separate projects without getting lost in all the buffers. Switching to a perspective activates its window configuration, and when in a perspective, only its buffers are available.

(use-package perspective
  :bind
  ("C-x C-b" . persp-list-buffers)   ; or use a nicer switcher, see below
  :config
  (persp-mode))
PROJECTILE

(use-package projectile
  :config
  (projectile-global-mode 1))
REGISTERS

Emacs registers are compartments where you can save text, rectangles and positions for later use. Once you save text or a rectangle in a register, you can copy it into the buffer once or many times; once you save a position in a register, you can jump back to that position once or many times. The default GNU Emacs keybindings for these commands (with the exception of counsel-register) involves ‘C-x r’ followed by one or more other keys. I wanted to make this a little more user friendly, so I chose to replace the ‘C-x r’ part of the key chords with ‘SPC r’.

COMMAND	DESCRIPTION	KEYBINDING
copy-to-register	Copy to register	SPC r c
frameset-to-register	Frameset to register	SPC r f
insert-register	Insert contents of register	SPC r i
jump-to-register	Jump to register	SPC r j
list-registers	List registers	SPC r l
number-to-register	Number to register	SPC r n
counsel-register	Interactively choose a register	SPC r r
view-register	View a register	SPC r v
window-configuration-to-register	Window configuration to register	SPC r w
increment-register	Increment register	SPC r +
point-to-register	Point to register	SPC r SPC
(nvmap :prefix "SPC"
       "r c"   '(copy-to-register :which-key "Copy to register")
       "r f"   '(frameset-to-register :which-key "Frameset to register")
       "r i"   '(insert-register :which-key "Insert register")
       "r j"   '(jump-to-register :which-key "Jump to register")
       "r l"   '(list-registers :which-key "List registers")
       "r n"   '(number-to-register :which-key "Number to register")
       "r r"   '(counsel-register :which-key "Choose a register")
       "r v"   '(view-register :which-key "View a register")
       "r w"   '(window-configuration-to-register :which-key "Window configuration to register")
       "r +"   '(increment-register :which-key "Increment register")
       "r SPC" '(point-to-register :which-key "Point to register"))
SCROLLING

Emacs’ default scrolling is annoying because of the sudden half-page jumps. Also, I wanted to adjust the scrolling speed.

(setq scroll-conservatively 101) ;; value greater than 100 gets rid of half page jumping
(setq mouse-wheel-scroll-amount '(3 ((shift) . 3))) ;; how many lines at a time
(setq mouse-wheel-progressive-speed t) ;; accelerate scrolling
(setq mouse-wheel-follow-mouse 't) ;; scroll window under mouse
SHELLS

In my configs, all of my shells (bash, fish, zsh and the ESHELL) require my shel-color-scripts-git package to be installed. On Arch Linux, you can install it from the AUR. Otherwise, go to my shell-color-scripts repository on GitLab to get it.

Eshell

Eshell is an Emacs ‘shell’ that is written in Elisp.

(nvmap :prefix "SPC"
       "e h"   '(counsel-esh-history :which-key "Eshell history")
       "e s"   '(eshell :which-key "Eshell"))
‘eshell-syntax-highlighting’ – adds fish/zsh-like syntax highlighting.
‘eshell-rc-script’ – your profile for eshell; like a bashrc for eshell.
‘eshell-aliases-file’ – sets an aliases file for the eshell.
(use-package eshell-syntax-highlighting
  :after esh-mode
  :config
  (eshell-syntax-highlighting-global-mode +1))

(setq eshell-rc-script (concat user-emacs-directory "eshell/profile")
      eshell-aliases-file (concat user-emacs-directory "eshell/aliases")
      eshell-history-size 5000
      eshell-buffer-maximum-lines 5000
      eshell-hist-ignoredups t
      eshell-scroll-to-bottom-on-input t
      eshell-destroy-buffer-when-process-dies t
      eshell-visual-commands'("bash" "fish" "htop" "ssh" "top" "zsh"))
Vterm

Vterm is a terminal emulator within Emacs. The ‘shell-file-name’ setting sets the shell to be used in M-x shell, M-x term, M-x ansi-term and M-x vterm. By default, the shell is set to ‘fish’ but could change it to ‘bash’ or ‘zsh’ if you prefer.

(use-package vterm)
(setq shell-file-name "/bin/fish"
      vterm-max-scrollback 5000)
IDE SETUP

Lets turn emacs into a powerful IDE!

Python

(use-package better-defaults)
SPLITS AND WINDOW CONTROLS

(winner-mode 1)
(nvmap :prefix "SPC"
       ;; Window splits
       "w c"   '(evil-window-delete :which-key "Close window")
       "w n"   '(evil-window-new :which-key "New window")
       "w s"   '(evil-window-split :which-key "Horizontal split window")
       "w v"   '(evil-window-vsplit :which-key "Vertical split window")
       ;; Window motions
       "w h"   '(evil-window-left :which-key "Window left")
       "w j"   '(evil-window-down :which-key "Window down")
       "w k"   '(evil-window-up :which-key "Window up")
       "w l"   '(evil-window-right :which-key "Window right")
       "w w"   '(evil-window-next :which-key "Goto next window")
       ;; winner mode
       "w <left>"  '(winner-undo :which-key "Winner undo")
       "w <right>" '(winner-redo :which-key "Winner redo"))
THEME

We need a nice colorscheme.

(use-package doom-themes)
(setq doom-themes-enable-bold t    ; if nil, bold is universally disabled
      doom-themes-enable-italic t) ; if nil, italics is universally disabled
(load-theme 'doom-challenger-deep t)
STRAIGHT.EL

(defvar bootstrap-version)
(let ((bootstrap-file
       (expand-file-name "straight/repos/straight.el/bootstrap.el" user-emacs-directory))
      (bootstrap-version 5))
  (unless (file-exists-p bootstrap-file)
    (with-current-buffer
        (url-retrieve-synchronously
         "https://raw.githubusercontent.com/raxod502/straight.el/develop/install.el"
         'silent 'inhibit-cookies)
      (goto-char (point-max))
      (eval-print-last-sexp)))
  (load bootstrap-file nil 'nomessage))
Telegram - Telga

This is to install Telga, a Telegram client for Emacs.

(use-package telega
    :straight (telega :type git :host github :repo "zevlg/telega.el")
    :commands telega
    ;; :hook (telega-load . (lambda ()
    ;; 			 (interactive)
    ;; 			 (require 'ol-telega)))
    :bind
    (:map telega-root-mode-map
     ("A" . telega-account-switch))
    :custom
    (telega-use-docker nil)
    (telega-old-date-format "20%Y.0%M.%D")
    (telega-chat-input-markups '("org" nil "markdown1" "markdown2"))
    ;(telega-use-tracking-for '(any pin unread))
    ;(telega-emoji-use-images t)
    (telega-completing-read-function #'completing-read)
    (telega-msg-rainbow-title t)
    (telega-chat-fill-column 75)
    (telega-debug nil)
    (telega-appindicator-labels '("❶" "❷" "❸" "❹" "❺" "❻" "❼" "❽" "❾" "❿" "⓫" "⓬" "⓭" "⓮" "⓯" "⓰" "⓱" "⓲" "⓳" "⓴"))
    (telega-accounts `(("writh" telega-database-dir ,(concat telega-database-dir "/writh"))
                       )))
    :config
    ;(setq telega-user-use-avatars nil)
    (telega-appindicator-mode t)
    (telega-notifications-mode t)
    (add-hook 'telega-root-mode-hook (lambda () (toggle-truncate-lines 1)))
(setq telega-emoji-company-backend 'telega-company-emoji)

(defun my-telega-chat-mode ()
  (set (make-local-variable 'company-backends)
       (append (list telega-emoji-company-backend
                   'telega-company-username
                   'telega-company-hashtag)
             (when (telega-chat-bot-p telega-chatbuf--chat)
               '(telega-company-botcmd))))
  (company-mode 1))

(add-hook 'telega-chat-mode-hook 'my-telega-chat-mode)
(add-hook 'after-init-hook 'global-company-mode)
WHICH KEY

Which-key is a minor mode for Emacs that displays the key bindings following your currently entered incomplete command (a prefix) in a popup.

NOTE: Which-key has an annoying bug that in some fonts and font sizes, the bottom row in which key gets covered up by the modeline.

(use-package which-key
  :init
  (setq which-key-side-window-location 'bottom
        which-key-sort-order #'which-key-key-order-alpha
        which-key-sort-uppercase-first nil
        which-key-add-column-padding 1
        which-key-max-display-columns nil
        which-key-min-display-lines 6
        which-key-side-window-slot -10
        which-key-side-window-max-height 0.25
        which-key-idle-delay 0.8
        which-key-max-description-length 25
        which-key-allow-imprecise-window-fit t
        which-key-separator " → " ))
(which-key-mode)
RUNTIME PERFORMANCE

Dial the GC threshold back down so that garbage collection happens more frequently but in less time.

;; Make gc pauses faster by decreasing the threshold.
(setq gc-cons-threshold (* 2 1000 1000))
WRITEROOM MODE

A minor mode for Emacs that implements a distraction-free writing mode similar to the famous Writeroom editor for OS X.

(use-package writeroom-mode)
