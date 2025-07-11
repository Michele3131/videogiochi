# 7. Gestione Eccezioni e Validazione

> **Riferimenti teorici**: Questo capitolo implementa le strategie di gestione errori e validazione descritte in [Gestione Errori](../knowledge/05-gestione-errori.md)

## 7.1 Sistema di Gestione Eccezioni

> **Best Practice**: Un sistema di eccezioni strutturato migliora la manutenibilità del codice e fornisce messaggi di errore chiari agli utenti.

### 7.1.1 Eccezioni Custom

> **Pattern implementato**: Utilizziamo una gerarchia di eccezioni custom per categorizzare e gestire diversi tipi di errori in modo specifico.

**BaseException.java**:
```java
package mc.videogiochi.exception;

import lombok.Getter;

@Getter
public abstract class BaseException extends RuntimeException {
    
    private final String errorCode;
    private final Object[] args;
    
    protected BaseException(String message, String errorCode) {
        super(message);
        this.errorCode = errorCode;
        this.args = null;
    }
    
    protected BaseException(String message, String errorCode, Object... args) {
        super(message);
        this.errorCode = errorCode;
        this.args = args;
    }
    
    protected BaseException(String message, Throwable cause, String errorCode) {
        super(message, cause);
        this.errorCode = errorCode;
        this.args = null;
    }
    
    protected BaseException(String message, Throwable cause, String errorCode, Object... args) {
        super(message, cause);
        this.errorCode = errorCode;
        this.args = args;
    }
}
```

**BusinessException.java**:
```java
package mc.videogiochi.exception;

public class BusinessException extends BaseException {
    
    public BusinessException(String message) {
        super(message, "BUSINESS_ERROR");
    }
    
    public BusinessException(String message, String errorCode) {
        super(message, errorCode);
    }
    
    public BusinessException(String message, String errorCode, Object... args) {
        super(message, errorCode, args);
    }
    
    public BusinessException(String message, Throwable cause) {
        super(message, cause, "BUSINESS_ERROR");
    }
    
    public BusinessException(String message, Throwable cause, String errorCode) {
        super(message, cause, errorCode);
    }
}
```

**ResourceNotFoundException.java**:
```java
package mc.videogiochi.exception;

public class ResourceNotFoundException extends BaseException {
    
    public ResourceNotFoundException(String resourceName, String fieldName, Object fieldValue) {
        super(String.format("%s non trovato con %s: '%s'", resourceName, fieldName, fieldValue), 
              "RESOURCE_NOT_FOUND", resourceName, fieldName, fieldValue);
    }
    
    public ResourceNotFoundException(String resourceName, Long id) {
        super(String.format("%s non trovato con ID: %d", resourceName, id), 
              "RESOURCE_NOT_FOUND", resourceName, "id", id);
    }
    
    public ResourceNotFoundException(String message) {
        super(message, "RESOURCE_NOT_FOUND");
    }
    
    public ResourceNotFoundException(String message, String errorCode) {
        super(message, errorCode);
    }
}
```

**ValidationException.java**:
```java
package mc.videogiochi.exception;

import lombok.Getter;

import java.util.Map;

@Getter
public class ValidationException extends BaseException {
    
    private final Map<String, String> fieldErrors;
    
    public ValidationException(String message) {
        super(message, "VALIDATION_ERROR");
        this.fieldErrors = null;
    }
    
    public ValidationException(String message, Map<String, String> fieldErrors) {
        super(message, "VALIDATION_ERROR");
        this.fieldErrors = fieldErrors;
    }
    
    public ValidationException(String message, String errorCode) {
        super(message, errorCode);
        this.fieldErrors = null;
    }
    
    public ValidationException(String message, String errorCode, Map<String, String> fieldErrors) {
        super(message, errorCode);
        this.fieldErrors = fieldErrors;
    }
}
```

**SecurityException.java**:
```java
package mc.videogiochi.exception;

public class SecurityException extends BaseException {
    
    public SecurityException(String message) {
        super(message, "SECURITY_ERROR");
    }
    
    public SecurityException(String message, String errorCode) {
        super(message, errorCode);
    }
    
    public SecurityException(String message, Throwable cause) {
        super(message, cause, "SECURITY_ERROR");
    }
    
    public SecurityException(String message, Throwable cause, String errorCode) {
        super(message, cause, errorCode);
    }
}
```

**DuplicateResourceException.java**:
```java
package mc.videogiochi.exception;

public class DuplicateResourceException extends BaseException {
    
    public DuplicateResourceException(String resourceName, String fieldName, Object fieldValue) {
        super(String.format("%s già esistente con %s: '%s'", resourceName, fieldName, fieldValue), 
              "DUPLICATE_RESOURCE", resourceName, fieldName, fieldValue);
    }
    
    public DuplicateResourceException(String message) {
        super(message, "DUPLICATE_RESOURCE");
    }
    
    public DuplicateResourceException(String message, String errorCode) {
        super(message, errorCode);
    }
}
```

**ExternalServiceException.java**:
```java
package mc.videogiochi.exception;

public class ExternalServiceException extends BaseException {
    
    public ExternalServiceException(String serviceName, String message) {
        super(String.format("Errore servizio esterno %s: %s", serviceName, message), 
              "EXTERNAL_SERVICE_ERROR", serviceName);
    }
    
    public ExternalServiceException(String serviceName, String message, Throwable cause) {
        super(String.format("Errore servizio esterno %s: %s", serviceName, message), 
              cause, "EXTERNAL_SERVICE_ERROR", serviceName);
    }
    
    public ExternalServiceException(String message, String errorCode) {
        super(message, errorCode);
    }
    
    public ExternalServiceException(String message, Throwable cause, String errorCode) {
        super(message, cause, errorCode);
    }
}
```

### 7.1.2 Global Exception Handler

**GlobalExceptionHandler.java**:
```java
package mc.videogiochi.exception;

import lombok.extern.slf4j.Slf4j;
import mc.videogiochi.dto.response.ApiResponse;
import org.springframework.dao.DataIntegrityViolationException;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.http.converter.HttpMessageNotReadableException;
import org.springframework.security.access.AccessDeniedException;
import org.springframework.security.authentication.BadCredentialsException;
import org.springframework.security.authentication.DisabledException;
import org.springframework.security.authentication.LockedException;
import org.springframework.validation.FieldError;
import org.springframework.web.HttpMediaTypeNotSupportedException;
import org.springframework.web.HttpRequestMethodNotSupportedException;
import org.springframework.web.bind.MethodArgumentNotValidException;
import org.springframework.web.bind.MissingServletRequestParameterException;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestControllerAdvice;
import org.springframework.web.context.request.WebRequest;
import org.springframework.web.method.annotation.MethodArgumentTypeMismatchException;
import org.springframework.web.servlet.NoHandlerFoundException;

import jakarta.servlet.http.HttpServletRequest;
import jakarta.validation.ConstraintViolation;
import jakarta.validation.ConstraintViolationException;
import java.time.LocalDateTime;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@RestControllerAdvice
@Slf4j
public class GlobalExceptionHandler {
    
    // ==================== Custom Exceptions ====================
    
    @ExceptionHandler(ResourceNotFoundException.class)
    public ResponseEntity<ApiResponse<Void>> handleResourceNotFoundException(
            ResourceNotFoundException ex, HttpServletRequest request) {
        
        log.warn("Risorsa non trovata: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error(ex.getMessage());
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }
    
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ApiResponse<Void>> handleValidationException(
            ValidationException ex, HttpServletRequest request) {
        
        log.warn("Errore di validazione: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error(ex.getMessage(), ex.getFieldErrors());
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    @ExceptionHandler(BusinessException.class)
    public ResponseEntity<ApiResponse<Void>> handleBusinessException(
            BusinessException ex, HttpServletRequest request) {
        
        log.warn("Errore business logic: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error(ex.getMessage());
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    @ExceptionHandler(DuplicateResourceException.class)
    public ResponseEntity<ApiResponse<Void>> handleDuplicateResourceException(
            DuplicateResourceException ex, HttpServletRequest request) {
        
        log.warn("Risorsa duplicata: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error(ex.getMessage());
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.CONFLICT).body(response);
    }
    
    @ExceptionHandler(SecurityException.class)
    public ResponseEntity<ApiResponse<Void>> handleSecurityException(
            SecurityException ex, HttpServletRequest request) {
        
        log.error("Errore di sicurezza: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error("Errore di sicurezza");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(response);
    }
    
    @ExceptionHandler(ExternalServiceException.class)
    public ResponseEntity<ApiResponse<Void>> handleExternalServiceException(
            ExternalServiceException ex, HttpServletRequest request) {
        
        log.error("Errore servizio esterno: {}", ex.getMessage(), ex);
        
        ApiResponse<Void> response = ApiResponse.error(
            "Servizio temporaneamente non disponibile. Riprova più tardi.");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.SERVICE_UNAVAILABLE).body(response);
    }
    
    // ==================== Spring Security Exceptions ====================
    
    @ExceptionHandler(BadCredentialsException.class)
    public ResponseEntity<ApiResponse<Void>> handleBadCredentialsException(
            BadCredentialsException ex, HttpServletRequest request) {
        
        log.warn("Credenziali non valide: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error("Username o password non corretti");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(response);
    }
    
    @ExceptionHandler(DisabledException.class)
    public ResponseEntity<ApiResponse<Void>> handleDisabledException(
            DisabledException ex, HttpServletRequest request) {
        
        log.warn("Account disabilitato: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error("Account disabilitato");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(response);
    }
    
    @ExceptionHandler(LockedException.class)
    public ResponseEntity<ApiResponse<Void>> handleLockedException(
            LockedException ex, HttpServletRequest request) {
        
        log.warn("Account bloccato: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error("Account bloccato");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.UNAUTHORIZED).body(response);
    }
    
    @ExceptionHandler(AccessDeniedException.class)
    public ResponseEntity<ApiResponse<Void>> handleAccessDeniedException(
            AccessDeniedException ex, HttpServletRequest request) {
        
        log.warn("Accesso negato: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error("Accesso negato");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(response);
    }
    
    // ==================== Validation Exceptions ====================
    
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Void>> handleMethodArgumentNotValidException(
            MethodArgumentNotValidException ex, HttpServletRequest request) {
        
        log.warn("Errori di validazione nei parametri del metodo");
        
        Map<String, String> fieldErrors = new HashMap<>();
        for (FieldError error : ex.getBindingResult().getFieldErrors()) {
            fieldErrors.put(error.getField(), error.getDefaultMessage());
        }
        
        ApiResponse<Void> response = ApiResponse.error("Errori di validazione", fieldErrors);
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    @ExceptionHandler(ConstraintViolationException.class)
    public ResponseEntity<ApiResponse<Void>> handleConstraintViolationException(
            ConstraintViolationException ex, HttpServletRequest request) {
        
        log.warn("Violazione constraint di validazione");
        
        Map<String, String> fieldErrors = new HashMap<>();
        Set<ConstraintViolation<?>> violations = ex.getConstraintViolations();
        
        for (ConstraintViolation<?> violation : violations) {
            String fieldName = violation.getPropertyPath().toString();
            String message = violation.getMessage();
            fieldErrors.put(fieldName, message);
        }
        
        ApiResponse<Void> response = ApiResponse.error("Errori di validazione", fieldErrors);
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    // ==================== HTTP Exceptions ====================
    
    @ExceptionHandler(HttpRequestMethodNotSupportedException.class)
    public ResponseEntity<ApiResponse<Void>> handleHttpRequestMethodNotSupportedException(
            HttpRequestMethodNotSupportedException ex, HttpServletRequest request) {
        
        log.warn("Metodo HTTP non supportato: {}", ex.getMethod());
        
        String message = String.format("Metodo HTTP '%s' non supportato per questo endpoint", ex.getMethod());
        ApiResponse<Void> response = ApiResponse.error(message);
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.METHOD_NOT_ALLOWED).body(response);
    }
    
    @ExceptionHandler(HttpMediaTypeNotSupportedException.class)
    public ResponseEntity<ApiResponse<Void>> handleHttpMediaTypeNotSupportedException(
            HttpMediaTypeNotSupportedException ex, HttpServletRequest request) {
        
        log.warn("Media type non supportato: {}", ex.getContentType());
        
        ApiResponse<Void> response = ApiResponse.error("Tipo di contenuto non supportato");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.UNSUPPORTED_MEDIA_TYPE).body(response);
    }
    
    @ExceptionHandler(HttpMessageNotReadableException.class)
    public ResponseEntity<ApiResponse<Void>> handleHttpMessageNotReadableException(
            HttpMessageNotReadableException ex, HttpServletRequest request) {
        
        log.warn("Messaggio HTTP non leggibile: {}", ex.getMessage());
        
        ApiResponse<Void> response = ApiResponse.error("Formato del messaggio non valido");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    @ExceptionHandler(MissingServletRequestParameterException.class)
    public ResponseEntity<ApiResponse<Void>> handleMissingServletRequestParameterException(
            MissingServletRequestParameterException ex, HttpServletRequest request) {
        
        log.warn("Parametro richiesto mancante: {}", ex.getParameterName());
        
        String message = String.format("Parametro richiesto '%s' mancante", ex.getParameterName());
        ApiResponse<Void> response = ApiResponse.error(message);
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    @ExceptionHandler(MethodArgumentTypeMismatchException.class)
    public ResponseEntity<ApiResponse<Void>> handleMethodArgumentTypeMismatchException(
            MethodArgumentTypeMismatchException ex, HttpServletRequest request) {
        
        log.warn("Tipo di argomento non corrispondente: {}", ex.getName());
        
        String message = String.format("Parametro '%s' deve essere di tipo %s", 
                                     ex.getName(), ex.getRequiredType().getSimpleName());
        ApiResponse<Void> response = ApiResponse.error(message);
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.BAD_REQUEST).body(response);
    }
    
    @ExceptionHandler(NoHandlerFoundException.class)
    public ResponseEntity<ApiResponse<Void>> handleNoHandlerFoundException(
            NoHandlerFoundException ex, HttpServletRequest request) {
        
        log.warn("Endpoint non trovato: {} {}", ex.getHttpMethod(), ex.getRequestURL());
        
        ApiResponse<Void> response = ApiResponse.error("Endpoint non trovato");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.NOT_FOUND).body(response);
    }
    
    // ==================== Database Exceptions ====================
    
    @ExceptionHandler(DataIntegrityViolationException.class)
    public ResponseEntity<ApiResponse<Void>> handleDataIntegrityViolationException(
            DataIntegrityViolationException ex, HttpServletRequest request) {
        
        log.error("Violazione integrità dati: {}", ex.getMessage());
        
        String message = "Operazione non consentita: violazione vincoli di integrità";
        
        // Analizza il tipo di violazione per fornire messaggi più specifici
        if (ex.getMessage() != null) {
            if (ex.getMessage().contains("Duplicate entry")) {
                message = "Risorsa già esistente";
            } else if (ex.getMessage().contains("foreign key constraint")) {
                message = "Operazione non consentita: risorsa referenziata da altri elementi";
            } else if (ex.getMessage().contains("cannot be null")) {
                message = "Campi obbligatori mancanti";
            }
        }
        
        ApiResponse<Void> response = ApiResponse.error(message);
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.CONFLICT).body(response);
    }
    
    // ==================== Generic Exception ====================
    
    @ExceptionHandler(Exception.class)
    public ResponseEntity<ApiResponse<Void>> handleGenericException(
            Exception ex, HttpServletRequest request) {
        
        log.error("Errore interno del server: {}", ex.getMessage(), ex);
        
        ApiResponse<Void> response = ApiResponse.error(
            "Errore interno del server. Contattare l'amministratore.");
        response.setPath(request.getRequestURI());
        
        return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(response);
    }
}
```

## 7.2 Sistema di Validazione

### 7.2.1 Validatori Custom

**@ValidPassword**:
```java
package mc.videogiochi.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = PasswordValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidPassword {
    
    String message() default "Password non valida";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};
    
    int minLength() default 8;
    
    int maxLength() default 128;
    
    boolean requireUppercase() default true;
    
    boolean requireLowercase() default true;
    
    boolean requireDigit() default true;
    
    boolean requireSpecialChar() default true;
}
```

**PasswordValidator.java**:
```java
package mc.videogiochi.validation;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;
import org.springframework.util.StringUtils;

import java.util.regex.Pattern;

public class PasswordValidator implements ConstraintValidator<ValidPassword, String> {
    
    private int minLength;
    private int maxLength;
    private boolean requireUppercase;
    private boolean requireLowercase;
    private boolean requireDigit;
    private boolean requireSpecialChar;
    
    private static final Pattern UPPERCASE_PATTERN = Pattern.compile("[A-Z]");
    private static final Pattern LOWERCASE_PATTERN = Pattern.compile("[a-z]");
    private static final Pattern DIGIT_PATTERN = Pattern.compile("[0-9]");
    private static final Pattern SPECIAL_CHAR_PATTERN = Pattern.compile("[!@#$%^&*()_+\\-=\\[\\]{};':,.<>?]");
    
    @Override
    public void initialize(ValidPassword constraintAnnotation) {
        this.minLength = constraintAnnotation.minLength();
        this.maxLength = constraintAnnotation.maxLength();
        this.requireUppercase = constraintAnnotation.requireUppercase();
        this.requireLowercase = constraintAnnotation.requireLowercase();
        this.requireDigit = constraintAnnotation.requireDigit();
        this.requireSpecialChar = constraintAnnotation.requireSpecialChar();
    }
    
    @Override
    public boolean isValid(String password, ConstraintValidatorContext context) {
        if (!StringUtils.hasText(password)) {
            addConstraintViolation(context, "Password non può essere vuota");
            return false;
        }
        
        if (password.length() < minLength) {
            addConstraintViolation(context, 
                String.format("Password deve essere lunga almeno %d caratteri", minLength));
            return false;
        }
        
        if (password.length() > maxLength) {
            addConstraintViolation(context, 
                String.format("Password non può essere più lunga di %d caratteri", maxLength));
            return false;
        }
        
        if (requireUppercase && !UPPERCASE_PATTERN.matcher(password).find()) {
            addConstraintViolation(context, "Password deve contenere almeno una lettera maiuscola");
            return false;
        }
        
        if (requireLowercase && !LOWERCASE_PATTERN.matcher(password).find()) {
            addConstraintViolation(context, "Password deve contenere almeno una lettera minuscola");
            return false;
        }
        
        if (requireDigit && !DIGIT_PATTERN.matcher(password).find()) {
            addConstraintViolation(context, "Password deve contenere almeno un numero");
            return false;
        }
        
        if (requireSpecialChar && !SPECIAL_CHAR_PATTERN.matcher(password).find()) {
            addConstraintViolation(context, "Password deve contenere almeno un carattere speciale");
            return false;
        }
        
        return true;
    }
    
    private void addConstraintViolation(ConstraintValidatorContext context, String message) {
        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(message).addConstraintViolation();
    }
}
```

**@ValidEmail**:
```java
package mc.videogiochi.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = EmailValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidEmail {
    
    String message() default "Formato email non valido";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};
    
    boolean allowEmpty() default false;
}
```

**EmailValidator.java**:
```java
package mc.videogiochi.validation;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;
import org.springframework.util.StringUtils;

import java.util.regex.Pattern;

public class EmailValidator implements ConstraintValidator<ValidEmail, String> {
    
    private boolean allowEmpty;
    
    private static final Pattern EMAIL_PATTERN = Pattern.compile(
        "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$"
    );
    
    @Override
    public void initialize(ValidEmail constraintAnnotation) {
        this.allowEmpty = constraintAnnotation.allowEmpty();
    }
    
    @Override
    public boolean isValid(String email, ConstraintValidatorContext context) {
        if (!StringUtils.hasText(email)) {
            return allowEmpty;
        }
        
        return EMAIL_PATTERN.matcher(email).matches();
    }
}
```

**@ValidReleaseDate**:
```java
package mc.videogiochi.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = ReleaseDateValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidReleaseDate {
    
    String message() default "Data di rilascio non valida";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};
    
    boolean allowFuture() default true;
    
    int maxYearsInPast() default 50;
    
    int maxYearsInFuture() default 5;
}
```

**ReleaseDateValidator.java**:
```java
package mc.videogiochi.validation;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

import java.time.LocalDate;

public class ReleaseDateValidator implements ConstraintValidator<ValidReleaseDate, LocalDate> {
    
    private boolean allowFuture;
    private int maxYearsInPast;
    private int maxYearsInFuture;
    
    @Override
    public void initialize(ValidReleaseDate constraintAnnotation) {
        this.allowFuture = constraintAnnotation.allowFuture();
        this.maxYearsInPast = constraintAnnotation.maxYearsInPast();
        this.maxYearsInFuture = constraintAnnotation.maxYearsInFuture();
    }
    
    @Override
    public boolean isValid(LocalDate releaseDate, ConstraintValidatorContext context) {
        if (releaseDate == null) {
            return true; // Lascia che @NotNull gestisca i valori null
        }
        
        LocalDate now = LocalDate.now();
        LocalDate minDate = now.minusYears(maxYearsInPast);
        LocalDate maxDate = now.plusYears(maxYearsInFuture);
        
        if (releaseDate.isBefore(minDate)) {
            addConstraintViolation(context, 
                String.format("Data di rilascio non può essere più di %d anni nel passato", maxYearsInPast));
            return false;
        }
        
        if (!allowFuture && releaseDate.isAfter(now)) {
            addConstraintViolation(context, "Data di rilascio non può essere nel futuro");
            return false;
        }
        
        if (allowFuture && releaseDate.isAfter(maxDate)) {
            addConstraintViolation(context, 
                String.format("Data di rilascio non può essere più di %d anni nel futuro", maxYearsInFuture));
            return false;
        }
        
        return true;
    }
    
    private void addConstraintViolation(ConstraintValidatorContext context, String message) {
        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(message).addConstraintViolation();
    }
}
```

**@ValidRating**:
```java
package mc.videogiochi.validation;

import jakarta.validation.Constraint;
import jakarta.validation.Payload;

import java.lang.annotation.*;

@Documented
@Constraint(validatedBy = RatingValidator.class)
@Target({ElementType.FIELD, ElementType.PARAMETER})
@Retention(RetentionPolicy.RUNTIME)
public @interface ValidRating {
    
    String message() default "Valutazione non valida";
    
    Class<?>[] groups() default {};
    
    Class<? extends Payload>[] payload() default {};
    
    double minValue() default 0.0;
    
    double maxValue() default 10.0;
    
    int decimalPlaces() default 1;
}
```

**RatingValidator.java**:
```java
package mc.videogiochi.validation;

import jakarta.validation.ConstraintValidator;
import jakarta.validation.ConstraintValidatorContext;

import java.math.BigDecimal;
import java.math.RoundingMode;

public class RatingValidator implements ConstraintValidator<ValidRating, Double> {
    
    private double minValue;
    private double maxValue;
    private int decimalPlaces;
    
    @Override
    public void initialize(ValidRating constraintAnnotation) {
        this.minValue = constraintAnnotation.minValue();
        this.maxValue = constraintAnnotation.maxValue();
        this.decimalPlaces = constraintAnnotation.decimalPlaces();
    }
    
    @Override
    public boolean isValid(Double rating, ConstraintValidatorContext context) {
        if (rating == null) {
            return true; // Lascia che @NotNull gestisca i valori null
        }
        
        if (rating < minValue || rating > maxValue) {
            addConstraintViolation(context, 
                String.format("Valutazione deve essere tra %.1f e %.1f", minValue, maxValue));
            return false;
        }
        
        // Verifica il numero di decimali
        BigDecimal bd = BigDecimal.valueOf(rating);
        int actualDecimalPlaces = bd.scale();
        
        if (actualDecimalPlaces > decimalPlaces) {
            addConstraintViolation(context, 
                String.format("Valutazione può avere al massimo %d cifre decimali", decimalPlaces));
            return false;
        }
        
        return true;
    }
    
    private void addConstraintViolation(ConstraintValidatorContext context, String message) {
        context.disableDefaultConstraintViolation();
        context.buildConstraintViolationWithTemplate(message).addConstraintViolation();
    }
}
```

### 7.2.2 Gruppi di Validazione

**ValidationGroups.java**:
```java
package mc.videogiochi.validation;

public class ValidationGroups {
    
    public interface Create {}
    
    public interface Update {}
    
    public interface Delete {}
    
    public interface Admin {}
    
    public interface User {}
    
    public interface Public {}
    
    public interface Registration {}
    
    public interface Login {}
    
    public interface PasswordChange {}
    
    public interface EmailVerification {}
    
    public interface PasswordReset {}
}
```

### 7.2.3 Configurazione Validazione

**ValidationConfig.java**:
```java
package mc.videogiochi.config;

import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.validation.beanvalidation.LocalValidatorFactoryBean;
import org.springframework.validation.beanvalidation.MethodValidationPostProcessor;

import jakarta.validation.Validator;

@Configuration
public class ValidationConfig {
    
    @Bean
    public Validator validator() {
        return new LocalValidatorFactoryBean();
    }
    
    @Bean
    public MethodValidationPostProcessor methodValidationPostProcessor() {
        MethodValidationPostProcessor processor = new MethodValidationPostProcessor();
        processor.setValidator(validator());
        return processor;
    }
}
```

### 7.2.4 Utility per Validazione

**ValidationUtils.java**:
```java
package mc.videogiochi.util;

import mc.videogiochi.exception.ValidationException;
import org.springframework.stereotype.Component;

import jakarta.validation.ConstraintViolation;
import jakarta.validation.Validator;
import java.util.HashMap;
import java.util.Map;
import java.util.Set;

@Component
public class ValidationUtils {
    
    private final Validator validator;
    
    public ValidationUtils(Validator validator) {
        this.validator = validator;
    }
    
    public <T> void validateAndThrow(T object, Class<?>... groups) {
        Set<ConstraintViolation<T>> violations = validator.validate(object, groups);
        
        if (!violations.isEmpty()) {
            Map<String, String> fieldErrors = new HashMap<>();
            
            for (ConstraintViolation<T> violation : violations) {
                String fieldName = violation.getPropertyPath().toString();
                String message = violation.getMessage();
                fieldErrors.put(fieldName, message);
            }
            
            throw new ValidationException("Errori di validazione", fieldErrors);
        }
    }
    
    public <T> Map<String, String> validate(T object, Class<?>... groups) {
        Set<ConstraintViolation<T>> violations = validator.validate(object, groups);
        Map<String, String> fieldErrors = new HashMap<>();
        
        for (ConstraintViolation<T> violation : violations) {
            String fieldName = violation.getPropertyPath().toString();
            String message = violation.getMessage();
            fieldErrors.put(fieldName, message);
        }
        
        return fieldErrors;
    }
    
    public <T> boolean isValid(T object, Class<?>... groups) {
        Set<ConstraintViolation<T>> violations = validator.validate(object, groups);
        return violations.isEmpty();
    }
    
    public static boolean isValidEmail(String email) {
        if (email == null || email.trim().isEmpty()) {
            return false;
        }
        
        String emailRegex = "^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\\.[a-zA-Z]{2,}$";
        return email.matches(emailRegex);
    }
    
    public static boolean isValidUsername(String username) {
        if (username == null || username.trim().isEmpty()) {
            return false;
        }
        
        // Username deve essere lungo 3-30 caratteri, solo lettere, numeri e underscore
        String usernameRegex = "^[a-zA-Z0-9_]{3,30}$";
        return username.matches(usernameRegex);
    }
    
    public static boolean isStrongPassword(String password) {
        if (password == null || password.length() < 8) {
            return false;
        }
        
        boolean hasUpper = password.chars().anyMatch(Character::isUpperCase);
        boolean hasLower = password.chars().anyMatch(Character::isLowerCase);
        boolean hasDigit = password.chars().anyMatch(Character::isDigit);
        boolean hasSpecial = password.chars().anyMatch(ch -> "!@#$%^&*()_+-=[]{};':,.<>?".indexOf(ch) >= 0);
        
        return hasUpper && hasLower && hasDigit && hasSpecial;
    }
}
```

Questo documento completa la gestione delle eccezioni e la validazione, fornendo un sistema robusto per la gestione degli errori e la validazione dei dati in input, essenziale per mantenere l'integrità e la sicurezza dell'applicazione.