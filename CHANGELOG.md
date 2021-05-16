# 2021/05/16: 0.2.2

- `LexerError` type now implements `Clone` and `Copy`.

# 2021/05/06: 0.2.1

- Fixed various bugs in `_` pattern handling.

# 2021/05/05: 0.2.0

- It is now possible to use the special lifetime `'input` in your token types
  to borrow from the input string. Example:

  ```rust
  enum Token<'input> {
      Id(&'input str),
  }

  lexer! {
      Lexer -> Token<'input>;

      rule Init {
          [' ' '\t' '\n']; // skip whitespace

          ['a'-'z']+ => |lexer| {
              let match_ = lexer.match_();
              lexer.return_(Token::Id(match_))
          },
      }
  }
  ```

  See also the Lua 5.1 lexer example, which is updated to use this feature.

- The `rule Init { ... }` syntax can now be omitted when you don't need named
  rule sets. For example, the example in the previous changelog entry can be
  simplified as:

  ```rust
  lexer! {
      Lexer -> Token<'input>;

      [' ' '\t' '\n'], // skip whitespace

      ['a'-'z']+ => |lexer| {
          let match_ = lexer.match_();
          lexer.return_(Token::Id(match_))
      },
  }
  ```

- `pub` keyword before a lexer name now generates the type as `pub`. Useful for
  using the generated lexer in other modules. Example:

  ```rust
  lexer! {
      pub Lexer -> Token;

      ...
  }
  ```

- Two new action kinds: "fallible" and "simple" added. The old ones defined
  with `=>` are now called "infallible".

  - "fallible" actions are defined with `=?` instead of `=>`. The difference
    from infallible actions is the return type is `Result<Token, UserError>`,
    instead of `Token`, where `UserError` is defined using `type Error = ...;`
    syntax. LHS can have a `<'input>` lifetime parameter when borrowing from
    the user input in the error values. When a user error type is defined, the
    lexer error struct becomes an enum, with two variants:

    ```rust
    enum LexerError {
        LexerError { char_idx: usize },
        UserError(UserError),
    }
    ```

  - "simple" actions are defined with `=` instead of `=>`. The RHS needs to be a
    value for a token, instead of a closure for a lexer action. This rule kind is
    useful when matching keywords and other simple tokens in a language. Example:

    ```rust
    lexer! {
        Lexer -> Token;

        '(' = Token::LParen,
        ')' = Token::RParen,
        ...
    }
    ```

    The syntax `<regex> = <expr>` is syntactic sugar for `<regex> => |lexer|
    lexer.return_(<expr>)`.

# 2021/04/22: 0.1.0

- Initial release