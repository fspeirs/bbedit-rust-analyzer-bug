# bbedit-rust-analyzer-bug

Repository to demonstrate a potential issue in either BBEdit or rust-analyzer.

I noticed an issue in BBEdit when using rust-analyzer when I wanted to change L1 in bbedit_test::main.rs.

use bbedit_lib::add;

becomes:

use bbedit_lib::{add, bbedit_do_thing};

I can reproduce the bug by:

- Select the text "add" in the import line.
- Type { to wrap it in braces
- Left arrow to move to after the opening brace
- Type-select "bbedi" to import "bbedit_do_thing"
- Tab to accept the completion.

Expected result: that the braces would contain `{bbedit_do_thing, add}`

Result: `add` is overwritten with `bbedit_do_thing`.

Feedback from BareBones tech support: 

When requesting the completion, BBEdit sends the following position parameter in the request:

```
    position =     {
        character = 22;
        line = 0;
    };
```

Character position 22 corresponds to where the insertion point is after typing "bbedi" just inside the open brace.

The response from the server includes an entry for `bbedit_do_thing`:
```
    {
        additionalTextEdits = (
        );
        deprecated = 0;
        detail = "fn()";
        filterText = "bbedit_do_thing";
        kind = 3;
        label = "bbedit_do_thing";
        preselect = 1;
        sortText = ffffffef;
        textEdit =             {
            newText = "bbedit_do_thing";
            range =                 {
                end =                     {
                    character = 25;
                    line = 0;
                };
                start =                     {
                    character = 17;
                    line = 0;
                };
            };
        };
   }
```

The `textEdit` value is an instruction to replace characters 17 through 25 on the first line with "bbedit_do_thing". This range includes the "add" that was already there.

When BBEdit follows the instruction specified by the server, the practical effect is to replace the `add` with `bbedit_do_thing`.
