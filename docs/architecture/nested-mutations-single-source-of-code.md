# Using a single source of code for standard and nested mutations

The GraphQL server supports 2 behaviors:

1. The standard behavior for mutations, by default
2. Nested mutations, as an opt-in

As a consequence, it will expose types `QueryRoot` and `MutationRoot` by default, and switch to exposing a single `Root` type for nested mutations.

When providing the resolvers, we wouldn't want to provide two resolvers, one for each solution. It's better that the same resolver used to resolve fields for `Root` can also resolve fields from `QueryRoot` and `MutationRoot`.

## Implementation details

The server uses an object called `FieldResolver` to resolve fields, and an object called `MutationResolver` to execute the actual mutation. The same `MutationResolver` object can be referenced by different `FieldResolvers` implementing different fields, so the code is implemented only once and used in many places, following the [SOLID](./solid.html) approach.

We know if a field is a mutation or not if the `FieldResolver` declares a `MutationResolver` object for that field, done through function `resolveFieldMutationResolverClass`.

For instance, field `Root.replyComment` [provides object `AddCommentToCustomPostMutationResolver`](https://github.com/PoPSchema/comment-mutations/blob/a5e1b03a8ca3f7723d990ba6d63c19b7f32c8c8d/src%2FFieldResolvers%2FRootFieldResolver.php#L65). This same object [is also used](https://github.com/PoPSchema/comment-mutations/blob/a5e1b03a8ca3f7723d990ba6d63c19b7f32c8c8d/src%2FFieldResolvers%2FCommentFieldResolver.php#L96) by field `Comment.reply`.

Furthermore, when coding the `FieldResolver`, the root fields are [added to the `Root` type](https://github.com/PoPSchema/comment-mutations/blob/a5e1b03a8ca3f7723d990ba6d63c19b7f32c8c8d/src%2FFieldResolvers%2FRootFieldResolver.php#L21) only. For the standard GraphQL behavior, the server can retrieve this configuration, and automatically add these fields to either `MutationRoot` or `QueryRoot` depending on if [they are mutations](https://github.com/GraphQLByPoP/graphql-server/blob/ece4d883a7e720ebb51f07b84efa94c0bbaa243c/src%2FTypeResolvers%2FMutationRootTypeResolver.php#L42) or [not](https://github.com/GraphQLByPoP/graphql-server/blob/a26eed6e6fa9facc7c56e35b666113d834529d4a/src%2FTypeResolvers%2FQueryRootTypeResolver.php#L42). 

As a result, since we are using a single source for the code powering both the standard behavior and nested mutations, we are able to execute queries with nested mutations for no extra effort.
