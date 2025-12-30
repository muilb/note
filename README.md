# note
## class

import com.fasterxml.jackson.annotation.JsonProperty;
import java.time.LocalDate;

public class UserRequest {

    @JsonProperty("user_name")
    private String userName;

    @JsonProperty("created_at")
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

