# Prompt: Generate an agent coding instruction

You will be given:
❌ Bad code (what must not be done)
✅ Good code (the expected pattern)
Your task is to generate a single coding instruction that captures the rule demonstrated by the examples.

## Rules for the instruction
- Write one rule only.
- Use imperative language (“Do not…”, “Always…”, “Prefer…”).
- Be specific to Rails/Ruby/RSpec as applicable.
- Make the rule clear and enforceable.
- Do not include explanations or rationale.
- Include code blocks showing the bad and good examples.
- Do not add any extra commentary.

## Output format
A short markdown heading or bullet containing the instruction.
A ❌ Bad code block.
A ✅ Good code block.

## Example output
One expectation per RSpec example — Do not write RSpec examples with more than one expect; split assertions into separate it blocks.

❌ Bad
```ruby
it 'returns groceries and tokens' do
  expect(groceries).to eq(payload)
  expect(tokens).to eq(count)
end
```

✅ Good
```ruby
it 'returns groceries' do
  expect(groceries).to eq(payload)
end

it 'returns tokens' do
  expect(tokens).to eq(count)
end
```