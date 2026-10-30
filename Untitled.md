query GetModelMetadata($environmentId: BigInt!) {
  environment(id: $environmentId) {
    applied {
      models(
        first: 10
        filter: {
          uniqueIds: "model.YOUR_PROJECT.team_a_cdo_positions"
        }
      ) {
        edges {
          node {
            name
            uniqueId
            description
            database
            schema
            alias
            materializedType
            contractEnforced
            catalog {
              columns {
                name
                type
              }
            }
          }
        }
      }
    }
  }
}