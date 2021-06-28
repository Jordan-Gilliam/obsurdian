# rust sandbox
 go to `/tmp` directory, create a new rust project, open Vim, run `cargo watch -x run`
 
 >  shell script that relies on `tmux`, `cargo-watch` and `nvim`.
 ```bash
#!/bin/sh
PLAYGROUNDS_DIR="/tmp/rust-playgrounds" 
TIMESTAMP=$(date +"%Y%m%d%H%M%S")
PROJECT_DIR="${PLAYGROUNDS_DIR}/playground${TIMESTAMP}"

cargo new $PROJECT_DIR 
cd $PROJECT_DIR

# Add dependencies to Cargo.toml
for CRATE in $@; do
 	sed "/^\[dependencies\]/a $CRATE = \"*\"" -i Cargo.toml 
done

tmux new-session \; \
 send-keys "nvim ./src/main.rs" C-m \; \
 split-window -h \; \
 send-keys "cargo watch -s 'clear && cargo run -q'" C-m \; \
 select-pane -L;
 ```
#tip #rust