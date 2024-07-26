---
title: "[gsoc] Exploring Code Generation: Swift Syntax, Swift Mustache, and Swift OpenAPI"
date: 2024-07-25
categories: [iOS, development]
tags: [gsoc, swift, swift on server]
mermaid: true
---

## Introduction

The main goal here is transforming [JSON Schema](https://github.com/aws/serverless-application-model/blob/develop/samtranslator/validator/sam_schema/schema.json) to Swift: Generate Swift Structs Based on Complex JSON Schema Specifications:
The process involves reading a JSON Schema and generating Swift structs that conform to the detailed specifications of the given JSON Schema.
To reach our objectives, we evaluated three Swift code generation techniques: Mustache templates, the OpenAPI Swift generator, and the swift Syntax library. This article provides a brief overview of each technique and proposes a solution for use in this project as part of google summer of code.

---

## Swift Mustache

> Mustache is a "logic-less" templating engine. The core language has no flow control statements. Instead it has tags that can be replaced with a value, nothing or a series of values.
> 

→ [Repo on Github](https://github.com/hummingbird-project/swift-mustache)
 How it works?   

The Swift Hummingbird Mustache implementation comes with a very simple Key-Value coding implementation, which is used to extract values from model objects for rendering. With that, you can render both generic Dictionary/Array structures as well as Swift objects with properties.

 Code Example:   
Generating a Codable struct with a minimal struct [template](https://github.com/esraaeiid/swift-aws-lambda-sam-dsl/blob/week-25/Sources/AWSLambdaDeploymentDescriptorGenerator/Templates/struct.swift):

```swift
struct Hello: Codable {
     let typeName: String
     let id: Int
     let properties: [Property]
     init(typeName: String, id: Int, properties: [Property]) {
        self.typeName = typeName
        self.id = id
        self.properties = properties
    }
     init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.typeName = try container.decode(String.self, forKey: .typeName)
        self.id = try container.decode(Int.self, forKey: .id)
        self.properties = try container.decode([Property].self, forKey: .properties)
    }
    private enum CodingKeys: String, CodingKey {
        case typeName
        case id
        case properties
    }
     struct Property: Codable {}
}
```

 Benefits:   

- Customizable templates with a separation of code generation templates from the core code, along with [examples](https://mustache.github.io/mustache.5.html) for multiple cases.
- Supports custom inheritance, optionals, and encoding/decoding.

 Challenges:   

- Mustache templates lack detailed debugging information, making it difficult to identify missing elements or required fixes.
- No comprehensive documentation was found that showcases various template examples.
- Mustache is a templating language that requires a learning curve for generating complex code, such as manual decoding and encoding, methods, statements, and enums with associated values.

 Important Links:   

- Mustache Templates Examples: [Link](https://docs.getdrafts.com/docs/actions/templates/mustache)
- Templating with Swift-Mustache: [Article](https://swiftonserver.com/templating-with-swift-mustache/)
- Mustache Documentation: [Github Pages](https://mustache.github.io/mustache.5.html)
- Mustache Syntax Documentation: [Github](https://github.com/hummingbird-project/swift-mustache/tree/main/documentation)

---

## Swift Syntax

> The swift-syntax package is a set of libraries that work on a source-accurate tree representation of Swift source code, called the SwiftSyntax tree.
> 

→ [Repo on Github](https://github.com/apple/swift-syntax)
 How it works?   

SwiftSyntax offers an accurate tree-based model of Swift source code, empowering Swift tools to parse, inspect, generate, and transform Swift source code using structured syntax defined in [SwiftSyntax](https://github.com/apple/swift-syntax/tree/main/Sources/SwiftSyntax) and [SwiftSyntaxBuilder](https://github.com/apple/swift-syntax/tree/main/Sources/SwiftSyntaxBuilder), with support from Swift [AST](https://swift-ast-explorer.com/) Explorer.

 Code Example:   
Generating a Codable struct using SwiftSyntax and SwiftSyntaxBuilder to [describe](https://github.com/esraaeiid/swift-aws-lambda-sam-dsl/blob/0be9dde63ab3145c3173386ba432587a583e030b/Sources/AWSLambdaDeploymentDescriptorGenerator/DeploymentDescriptorGenerator.swift#L69C10-L69C45) struct keys:

```swift

public struct Example: Decodable, Equatable {
    let typeName: String
    let properties: [Property]
    let subTypes: [Example]
    init(typeName: String, properties: [Property], subTypes: [Example]) {
        self.typeName = typeName
        self.properties = properties
        self.subTypes = subTypes
    }
    public init(from decoder: Decoder) throws {
        let container = try decoder.container(keyedBy: CodingKeys.self)
        self.typeName = try container.decode(String.self, forKey: .typeName)
        self.properties = try container.decode([Property].self, forKey: .properties)
        self.subTypes = try container.decode([Example].self, forKey: .subTypes)
    }
    public struct Property: Decodable, Equatable {
        let name: String?
        let type: String
    }
    private enum CodingKeys: String, CodingKey {
        case typeName = "typeName"
        case properties
        case subTypes
    }
}
```

[Generating](https://github.com/sebsto/swift-aws-lambda-sam-dsl/blob/22ba8e864ac67ea2bcb3164828c0a064f8298495/Sources/AWSLambdaDeploymentDescriptorGeneratorSwiftSyntax/SwiftDeploymentDescriptorGenerator.swift#L51) function and function statements:

```swift
   @discardableResult public func exampleFunctionFromBuilder(a: String, b: Bool) -> String {let foo = "Hello World"
        return foo
    }
    
    func fibonacci(_ n: Int) -> Int {
        if n <= 1 {
            return n
        }
        return fibonacci(n - 1) + self.fibonacci(n - 2)
    }
    
    func bar() async throws -> Int {
        var result = 0
        for elem in myArray {
            result += elem
        }
        return result
    }
```

 Benefits:   

- Easy composition and reusability of Swift code structures.
- It integrates with swift-parser and swift-format, ensuring compatibility and consistency in Swift code generation.
- SwiftSyntaxBuilder that provides powerful tools for constructing and manipulating Swift syntax trees.
- The [Swift AST Explorer](https://swift-ast-explorer.com/) has proven invaluable for comprehending the hierarchical structure required for generating Swift code.
- Enhancing flexibility and type safety in Swift models as It supports defining optional keys and enums with raw values for coding keys, also enable the creation of model objects with custom initializers crucial for decoding and initializing Swift objects from external representations.

 Challenges:   

- Requires careful planning and understanding of Swift's syntax and semantics.
- According to discussions in [this thread](https://forums.swift.org/t/announcing-swiftsyntaxbuilder/56565/6), generating functions or control statements using SwiftSyntaxBuilder may require additional effort and attention to detail. However it could be managed with [swift-syntax](https://github.com/sebsto/swift-aws-lambda-sam-dsl/blob/22ba8e864ac67ea2bcb3164828c0a064f8298495/Sources/AWSLambdaDeploymentDescriptorGeneratorSwiftSyntax/SwiftDeploymentDescriptorGenerator.swift#L51).
- There is a lack of comprehensive documentation that effectively showcases Swift Syntax Builder. However, there are numerous [test cases](https://github.com/apple/swift-syntax/tree/main/Tests/SwiftSyntaxBuilderTest) for Swift Syntax Builder that cover a wide range of syntax examples.
- Occasionally, debugging SwiftSyntax may present challenges such as vague error messages like "The compiler is unable to type-check this expression in reasonable time; try breaking up the expression into distinct sub-expressions” that doesn’t represent the real error.

 Important Links:   

- Swift AST explorer: [AST](https://swift-ast-explorer.com/)
- CodeGenerationUsingSwiftSyntaxBuilder: [Example](https://github.com/apple/swift-syntax/blob/main/Examples/Sources/CodeGenerationUsingSwiftSyntaxBuilder/CodeGenerationUsingSwiftSyntaxBuilder.swift)
- SwiftSyntax Documentation: [Doc](https://swiftpackageindex.com/apple/swift-syntax/508.0.1/documentation/swiftsyntax)
- Working with SwiftSyntax: [Github](https://github.com/apple/swift-syntax/blob/main/Sources/SwiftSyntax/Documentation.docc/Working%20with%20SwiftSyntax.md)
- Discussions Generating Code using SwiftSyntax: [Swift-Forums](https://forums.swift.org/search?q=space%20%23swift-users%20tags%3Aswift-syntax)
- SwiftSyntaxBuilder: [Swift-Forums](https://forums.swift.org/t/announcing-swiftsyntaxbuilder/56565)
- Examples using SwiftSyntaxBuilder: [Github](https://github.com/apple/swift-syntax/tree/main/Tests/SwiftSyntaxBuilderTest)
- SwiftSyntax: [Github](https://github.com/apple/swift-syntax/blob/main/Sources/SwiftSyntax/Documentation.docc/Tutorials/SwiftSyntax%20By%20Example.tutorial), [Doc](https://github.com/apple/swift-syntax/blob/main/Sources/SwiftSyntax/Documentation.docc/generated/SwiftSyntax.md#tutorials)
- SwiftSyntaxBuilder: [Github](https://github.com/apple/swift-syntax/tree/main/Sources/SwiftSyntaxBuilder)
- Example usage for Swift syntax: [Github](https://github.com/apple/swift-syntax/tree/main/Examples)
- Result Builder: [WWDC Session](https://developer.apple.com/wwdc21/10253), [article](https://www.avanderlee.com/swift/result-builders/)

---

## Swift OpenAPI

> OpenAPI is a specification for documenting HTTP services. An OpenAPI document is written in either YAML or JSON, and can be read by tools to help automate workflows
> 

→ [Repo on Github](https://github.com/apple/swift-openapi-generator)
 How it works?   

An OpenAPI document is written in either YAML or JSON, and can be read by tools to help automate workflows.

 Code Example:   

Generating Swift Code based on [yaml](https://github.com/sebsto/swift-aws-lambda-sam-dsl/blob/22ba8e864ac67ea2bcb3164828c0a064f8298495/Sources/AWSLambdaDeploymentDescriptorGeneratorOpenAPI/openapi.yaml) provided:

```swift
        /// - Remark: Generated from `#/components/schemas/AWSServerlessSimpleTable.ProvisionedThroughput`.
        internal struct AWSServerlessSimpleTable_period_ProvisionedThroughput: Codable, Hashable, Sendable {
            /// - Remark: Generated from `#/components/schemas/AWSServerlessSimpleTable.ProvisionedThroughput/ReadCapacityUnits`.
            internal var ReadCapacityUnits: Swift.Double?
            /// - Remark: Generated from `#/components/schemas/AWSServerlessSimpleTable.ProvisionedThroughput/WriteCapacityUnits`.
            internal var WriteCapacityUnits: Swift.Double
            /// Creates a new `AWSServerlessSimpleTable_period_ProvisionedThroughput`.
            ///
            /// - Parameters:
            ///   - ReadCapacityUnits:
            ///   - WriteCapacityUnits:
            internal init(
                ReadCapacityUnits: Swift.Double? = nil,
                WriteCapacityUnits: Swift.Double
            ) {
                self.ReadCapacityUnits = ReadCapacityUnits
                self.WriteCapacityUnits = WriteCapacityUnits
            }
            internal enum CodingKeys: String, CodingKey {
                case ReadCapacityUnits
                case WriteCapacityUnits
            }
            internal init(from decoder: any Decoder) throws {
                let container = try decoder.container(keyedBy: CodingKeys.self)
                ReadCapacityUnits = try container.decodeIfPresent(
                    Swift.Double.self,
                    forKey: .ReadCapacityUnits
                )
                WriteCapacityUnits = try container.decode(
                    Swift.Double.self,
                    forKey: .WriteCapacityUnits
                )
                try decoder.ensureNoAdditionalProperties(knownKeys: [
                    "ReadCapacityUnits",
                    "WriteCapacityUnits"
                ])
            }
        }
```

 Benefits:   

- One significant advantage is that it is already available, obviating the need to develop a new generator. The generated Swift code conforms to a structured format based on the OpenAPI specification, providing an efficient and straightforward solution for simple code generation.

 Challenges:   

- The generated Swift code conforms to a structured format defined by the OpenAPI specification. While it excels in adhering to Swift code generation best practices, it also face limitations in handling complex [JSON schemas](https://github.com/aws/serverless-application-model/blob/develop/samtranslator/validator/sam_schema/schema.json). Such as: nested structs, statement functions, and managing edge cases with enums.

 Important Links:   

- Swift OpenAPI Generator: [Tutorials](https://swiftpackageindex.com/apple/swift-openapi-generator/1.2.1/tutorials/swift-openapi-generator/clientswiftpm)
- Swift OpenAPI Generator: [Documentation](https://swiftpackageindex.com/apple/swift-openapi-generator/1.2.1/documentation/swift-openapi-generator)

---

## Conclusion

Based on community insights from [Swift-Forums](https://forums.swift.org/t/code-generation-swift-syntax-or-mustache/72131) and examples provided, I believe using swift-syntax is a more convenient option for generating Swift structs based on [complex JSON schema](https://github.com/aws/serverless-application-model/blob/develop/samtranslator/validator/sam_schema/schema.json) specifications.
The challenges of swift-syntax are manageable with careful planning and understanding of Swift's syntax and semantics. Debugging SwiftSyntax may present challenges such as vague error messages but can be managed through iterative code refinement and problem-solving strategies.
