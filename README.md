# note
## class

import com.fasterxml.jackson.databind.PropertyNamingStrategies;
import com.fasterxml.jackson.databind.annotation.JsonNaming;
@JsonNaming(PropertyNamingStrategies.SnakeCaseStrategy.class)
public class UserRequest {
    private String userName;
    private LocalDate createdAt;
}

## properties

import com.fasterxml.jackson.annotation.JsonProperty;
public class UserRequest {
    @JsonProperty("user_name")
    private String userName;

    @JsonProperty("created_at")
    private LocalDate createdAt;
}

