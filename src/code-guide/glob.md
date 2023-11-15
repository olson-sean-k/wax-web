## Basic Usage

Match a path against a glob:

``` { .rust }
use wax::{Glob, Pattern};

let glob = Glob::new("*.png").unwrap();
assert!(glob.is_match("logo.png"));
```

Match a path against a glob with matched text (captures):

``` { .rust }
use wax::{CandidatePath, Glob, Pattern};

let glob = Glob::new("**/{*.{go,rs}}").unwrap();

let path = CandidatePath::from("src/main.go");
let matched = glob.matched(&path).unwrap();

assert_eq!("main.go", matched.get(2).unwrap());
```

Match a directory tree against a glob:

``` { .rust }
use wax::Glob;

let glob = Glob::new("**/*.{md,txt}").unwrap();
for entry in glob.walk("doc") {
    let entry = entry.unwrap();
    // ...
}
```

Match a directory tree against a glob with negations:

``` { .rust }
use wax::{Glob, LinkBehavior};

let glob = Glob::new("**/*.{md,txt}").unwrap();
for entry in glob
    .walk_with_behavior("doc", LinkBehavior::ReadTarget)
    .not(["**/secret/**"])
    .unwrap()
{
    let entry = entry.unwrap();
    // ...
}
```

Match a path against multiple globs:

``` { .rust }
use wax::{Glob, Pattern};

let any = wax::any([
    "src/**/*.rs",
    "tests/**/*.rs",
    "doc/**/*.md",
    "pkg/**/PKGBUILD",
]).unwrap();
assert!(any.is_match("src/token/mod.rs"));
```
