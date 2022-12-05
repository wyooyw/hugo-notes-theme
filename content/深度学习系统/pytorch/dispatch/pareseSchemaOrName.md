---
linktitle: pareseSchemaOrName
summary: Summary of pareseSchemaOrName
type: book
---
# pareseSchemaOrName
打断点b torch::jit::parseSchemaOrName
torch/csrc/jit/frontend/function_schema_parser.cpp:373
```c++
either<OperatorName, FunctionSchema> parseSchemaOrName(
    const std::string& schemaOrName) {
  return SchemaParser(schemaOrName).parseExactlyOneDeclaration();
}
```

