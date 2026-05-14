---
name: Flutter Auth Feature Skill
description: Complete end-to-end Flutter auth skill — Social signin (Google/Apple), OTP screen, Create Profile form, Create Username form. Generates everything from one command: validators, formatters, date utils, dialog utils, BLoC, datasource, repository, usecase, screens, widgets, DI, routes.
type: project
---

# FLUTTER AUTH FEATURE SKILL (Global)

## HOW TO USE THIS SKILL

When user says "build auth", "implement login", "create auth feature":

**ASK THE USER ONLY:**
1. Auth method — Social (Google/Apple) only? or Email+Password? or OTP?
2. API endpoint for login — e.g., `/auth/login`
3. API response shape — e.g., `{ token, patient: {...}, isUsernameAdded, isUserDetailsFilled }`
4. Post-login flow — what screens to navigate to?
5. Does the app need profile creation after login? What fields?
6. OTP length — 4 or 5 or 6 digits?

**EVERYTHING ELSE IS GENERATED AUTOMATICALLY:**
- Validators (email, password, username, name, bio, OTP, phone, date)
- Date formatters (dd/MM/yyyy ↔ ISO8601)
- Dialog utils (calendar picker dialog)
- TextFieldComponent usage with proper focus management
- ButtonComponent usage
- Full auth BLoC (Google + Apple with device info)
- DataSource (abstract + impl) with RequestConfig headers
- Repository implementing both Google/Apple interfaces
- UseCase
- All screens: SignIn, OTP, CreateProfile, CreateUsername
- Hero logo animation
- RichText T&C + Privacy links with TapGestureRecognizer
- Platform-specific buttons (Platform.isAndroid / Platform.isIOS)
- loaderOverlay show/hide on state changes
- AwesomeNotifier.showSnackBar on failure
- Form with GlobalKey + FocusNode chain
- StorageService save on success
- DI registration
- Route enum entries
- GoRouter registration

---

## FULL AUTH FLOW

```
App Launch
    ↓
SocialSigninScreen (Google / Apple platform-specific)
    ↓ BLoC: SocialAuthLoading → SocialAuthSuccess
    ↓
Check isUserDetailsFilled?
    No  → CreateProfileScreen (firstName, lastName, email, dob, bio, profilePic)
    Yes → check isUsernameAdded?
              No  → CreateUsernameScreen
              Yes → HomeScreen
                    + ConversationsBloc.ConnectIfLoggedIn()
```

---

## FOLDER STRUCTURE

```
lib/features/auth/
├── data/
│   ├── datasources/
│   │   └── auth_data_source.dart        ← abstract + impl (same file)
│   ├── models/
│   │   └── social_signin_model.dart     ← extends entity directly
│   └── repositories/
│       └── auth_repository_impl.dart    ← implements both Google+Apple repos
├── domain/
│   ├── entities/
│   │   └── social_signin_entity.dart
│   ├── repositories/
│   │   ├── google_signin_repository.dart
│   │   └── apple_signin_repository.dart
│   └── usecases/
│       ├── google_signin_usecase.dart
│       └── apple_signin_usecase.dart
└── presentation/
    ├── bloc/
    │   ├── social_auth_bloc.dart         ← uses part of pattern
    │   ├── social_auth_event.dart        ← part of social_auth_bloc.dart
    │   └── social_auth_state.dart        ← part of social_auth_bloc.dart
    └── screens/
        ├── social_signin_screen.dart
        └── verify_otp_screen.dart        ← standalone widget

lib/features/user/presentation/screens/
    ├── create_profile_screen.dart        ← post-login profile setup
    └── create_username_screen.dart       ← post-profile username

lib/core/utils/
    ├── validations.dart                  ← Validation static class
    ├── date_utils.dart                   ← DateUtilsHelper static class
    ├── dialog_utils.dart                 ← DialogUtils static class
    └── awesome_notifier.dart             ← AwesomeNotifier static class

lib/core/params/
    └── login_params.dart                 ← SocialSigninParams
```

---

## STEP 1 — VALIDATORS (lib/core/utils/validations.dart)

All validators are static methods in `Validation` class.

```dart
class Validation {
  static String? validateEmail(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter your email';
    const pattern = r"^(?!\.)(?!.*\.\.)[a-zA-Z0-9._%+-]+(?<!\.)@[a-zA-Z0-9-]+(?:\.[a-zA-Z]{2,})+$";
    if (!RegExp(pattern).hasMatch(value.trim())) return 'Please enter a valid email address.';
    return null;
  }

  static String? validatePassword(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter your password';
    if (value.contains(' ')) return 'Password cannot contain white spaces';
    const pattern = r'^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*\W)[A-Za-z\d\W]{8,}$';
    if (!RegExp(pattern).hasMatch(value))
      return 'Password must be at least 8 characters long, include at least one uppercase letter, one lowercase letter, one number, and one special character';
    return null;
  }

  static String? confirmValidatePassword(String current, String newPass) {
    if (current.isEmpty || newPass.isEmpty) return 'Please enter new password';
    if (current != newPass) return 'Passwords do not match.';
    return null;
  }

  static String? validateUsername(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter a username';
    if (value != value.trim()) return 'Username cannot start or end with a space';
    final t = value.trim();
    if (t.length < 3 || t.length > 20) return 'Username must be between 3 and 20 characters';
    if (!RegExp(r'^[a-zA-Z0-9_]+$').hasMatch(t)) return 'Only letters, numbers, and underscores allowed';
    return null;
  }

  static String? validateFirstName(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter your first name';
    if (value != value.trim()) return 'First name cannot start or end with a space';
    if (!RegExp(r"^[a-zA-Z]+([-' ][a-zA-Z]+)*$").hasMatch(value)) return 'Use only letters';
    return null;
  }

  static String? validateLastName(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter your last name';
    if (value != value.trim()) return 'Last name cannot start or end with a space';
    if (!RegExp(r"^[a-zA-Z]+([-' ][a-zA-Z]+)*$").hasMatch(value)) return 'Use only letters';
    return null;
  }

  static String? validateBio(String? value) {
    if (value == null || value.trim().isEmpty) return 'Bio cannot be empty';
    if (value != value.trim()) return 'Bio cannot start or end with a space';
    final t = value.trim();
    if (t.length < 10) return 'Bio must be at least 10 characters';
    if (t.length > 150) return 'Bio cannot exceed 150 characters';
    return null;
  }

  static String? validateDateOfBirth(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter your Date Of Birth';
    return null;
  }

  static String? validateOtp(String? value) {
    if (value == null || value.trim().isEmpty) return 'Please enter OTP';
    if (value.trim().length != 5) return 'OTP must be 5 digits';    // change 5 to OTP length
    if (!RegExp(r'^\d+$').hasMatch(value.trim())) return 'OTP must contain only digits';
    return null;
  }

  static String? validateUSPhoneNumber(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter your phone number';
    if (!RegExp(r'^\d{10}$').hasMatch(value)) return 'Enter a valid phone number';
    return null;
  }

  static String? validateCommon(String? value) {
    if (value == null || value.trim().isEmpty) return "Field can't be empty";
    return null;
  }

  static String? validateCommentField(String? value) {
    if (value == null || value.trim().isEmpty) return 'Enter your comment';
    if (value != value.trim()) return 'Comment cannot start or end with a space';
    return null;
  }
}
```

---

## STEP 2 — DATE UTILS (lib/core/utils/date_utils.dart)

```dart
import 'package:intl/intl.dart';

class DateUtilsHelper {
  static String formatToDdMmYyyy(DateTime date) => DateFormat('dd/MM/yyyy').format(date);
  static String formatToYyyyMmDd(DateTime date) => DateFormat('yyyy-MM-dd').format(date);
  static String formatToReadable(DateTime date) => DateFormat('MMM dd, yyyy').format(date);
  static String formatIsoToDdMmYyyy(String isoString) => DateFormat('dd/MM/yyyy').format(DateTime.parse(isoString));
  static String formatIsoToDdMmYyyyLocal(String isoString) => DateFormat('dd/MM/yyyy').format(DateTime.parse(isoString).toLocal());
  static String toIsoString(DateTime date) => date.toIso8601String();

  // ← KEY: converts display format back to ISO for API
  static String formatDdMmYyyyToIso(String dateString) {
    final date = DateFormat('dd/MM/yyyy').parse(dateString);
    return date.toIso8601String();
  }
}
```

---

## STEP 3 — DIALOG UTILS (lib/core/utils/dialog_utils.dart)

```dart
class DialogUtils {
  static Future<void> showCalendarDialog(
    BuildContext context, {
    required Function(DateTime date) onDateSelected,
  }) async {
    await showDialog(
      context: context,
      builder: (_) => Dialog(
        backgroundColor: Colors.white,
        shape: RoundedRectangleBorder(borderRadius: BorderRadius.circular(16)),
        insetPadding: EdgeInsets.symmetric(horizontal: 20.w),
        child: ConstrainedBox(
          constraints: BoxConstraints(maxHeight: 400.h),
          child: Padding(
            padding: EdgeInsets.symmetric(vertical: 10.h),
            child: CustomCalender(
              onDayPress: (date) {
                onDateSelected(date);
                Navigator.of(context).pop();
              },
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## STEP 4 — PARAMS (lib/core/params/login_params.dart)

```dart
class SocialSigninParams {
  final String type;     // "google" or "apple"
  final String idToken;

  const SocialSigninParams({required this.type, required this.idToken});

  Map<String, String?> toJson() => {'type': type, 'idToken': idToken};
}
```

---

## STEP 5 — ENTITY (domain layer)

```dart
// social_signin_entity.dart — references UserEntity (shared)
class SocialSigninEntity {
  final String token;
  final UserEntity patient;         // ← domain user entity
  final bool isUsernameAdded;
  final bool isUserDetailsFilled;

  const SocialSigninEntity({
    required this.token,
    required this.patient,
    this.isUsernameAdded = false,
    this.isUserDetailsFilled = false,
  });
}
```

---

## STEP 6 — MODEL (data layer — EXTENDS ENTITY)

```dart
// Model extends Entity directly — no separate toEntity() needed
class SocialSignin extends SocialSigninEntity {
  const SocialSignin({
    required super.token,
    required User super.patient,   // ← User (data model) upcast to UserEntity
    super.isUsernameAdded = false,
    super.isUserDetailsFilled = false,
  });

  factory SocialSignin.fromJson(Map<String, dynamic> json) {
    return SocialSignin(
      token: json['token']?.toString() ?? '',
      patient: User.fromJson(json['patient'] ?? {}),
      isUsernameAdded: json['isUsernameAdded'] ?? false,
      isUserDetailsFilled: json['isUserDetailsFilled'] ?? false,
    );
  }

  Map<String, dynamic> toJson() => {
    'token': token,
    'patient': (patient as User).toJson(),
    'isUsernameAdded': isUsernameAdded,
    'isUserDetailsFilled': isUserDetailsFilled,
  };
}
```

---

## STEP 7 — DATASOURCE

```dart
abstract class AuthDatasource {
  Future<Either<Failure, SocialSignin>> signInWithGoogle(
    SocialSigninParams params, Map<String, dynamic> deviceInfo,
  );
  Future<Either<Failure, SocialSignin>> signInWithApple(
    SocialSigninParams params, Map<String, dynamic> deviceInfo,
  );
}

class AuthDatasourceImpl implements AuthDatasource {
  final DioApiService apiService = locator<DioApiService>();

  @override
  Future<Either<Failure, SocialSignin>> signInWithGoogle(
    SocialSigninParams params, Map<String, dynamic> deviceInfo,
  ) async {
    try {
      ApiResponse res = await apiService.post(
        ApiEndpoints.login,
        data: params,
        config: RequestConfig(headers: deviceInfo),   // ← device info as headers
      );
      return Right(SocialSignin.fromJson(res.data));
    } on ApiException catch (e) {
      debugPrint("login error: $e");
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      debugPrint("login error: $e");
      return Left(ServerFailure(message: 'Unexpected error: ${e.toString()}'));
    }
  }

  @override
  Future<Either<Failure, SocialSignin>> signInWithApple(
    SocialSigninParams params, Map<String, dynamic> deviceInfo,
  ) async {
    // same as google — same endpoint, different type param
    try {
      ApiResponse res = await apiService.post(
        ApiEndpoints.login,
        data: params,
        config: RequestConfig(headers: deviceInfo),
      );
      return Right(SocialSignin.fromJson(res.data));
    } on ApiException catch (e) {
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      return Left(ServerFailure(message: 'Unexpected error: ${e.toString()}'));
    }
  }
}
```

---

## STEP 8 — REPOSITORY (implements BOTH interfaces)

```dart
class AuthRepositoryImpl
    implements GoogleSigninRepository, AppleSigninRepository {
  final AuthDatasource dataSource;
  const AuthRepositoryImpl(this.dataSource);

  @override
  Future<Either<Failure, SocialSigninEntity>> signInWithGoogle(
    SocialSigninParams params, Map<String, dynamic> deviceInfo,
  ) async {
    final res = await dataSource.signInWithGoogle(params, deviceInfo);
    return res.fold(
      (left) => Left(ServerFailure(message: left.message)),
      (right) => Right(right),   // model extends entity — no conversion needed
    );
  }

  @override
  Future<Either<Failure, SocialSigninEntity>> signInWithApple(
    SocialSigninParams params, Map<String, dynamic> deviceInfo,
  ) async {
    final res = await dataSource.signInWithApple(params, deviceInfo);
    return res.fold(
      (left) => Left(ServerFailure(message: left.message)),
      (right) => Right(right),
    );
  }
}
```

---

## STEP 9 — BLOC (with device info helper)

```dart
// ── Top-level singleton (file scope)
final GoogleSignIn _googleSignIn = GoogleSignIn(
  scopes: ['email'],
  serverClientId: "YOUR_WEB_CLIENT_ID",
  clientId: "YOUR_IOS_CLIENT_ID",
);

class SocialAuthBloc extends Bloc<SocialAuthEvent, SocialAuthState> {
  final GoogleSigninUseCase googleSigninUseCase;
  final AppleSigninUseCase appleSigninUseCase;

  SocialAuthBloc(this.googleSigninUseCase, this.appleSigninUseCase)
      : super(SocialAuthInitial()) {

    // ── GOOGLE SIGN IN
    on<SignInWithGoogleRequested>((event, emit) async {
      emit(SocialAuthLoading());
      try {
        await _googleSignIn.signOut();        // force account picker always
        final account = await _googleSignIn.signIn();
        if (account == null) { emit(SocialAuthInitial()); return; }

        final auth = await account.authentication;
        final idToken = auth.idToken;

        if (idToken != null && idToken.isNotEmpty) {
          final deviceInfo = await _getDeviceInfo();
          final params = SocialSigninParams(type: "google", idToken: idToken);
          final result = await googleSigninUseCase.call(params, deviceInfo);

          result.fold(
            (failure) => emit(SocialAuthFailure(failure.message)),
            (res) {
              _saveSession(res);
              emit(SocialAuthSuccess(res));
            },
          );
        }
      } catch (e) {
        emit(SocialAuthFailure('Sign in failed: $e'));
      }
    });

    // ── APPLE SIGN IN
    on<SignInWithAppleRequested>((event, emit) async {
      try {
        emit(SocialAuthLoading());
        final credential = await SignInWithApple.getAppleIDCredential(
          scopes: [
            AppleIDAuthorizationScopes.email,
            AppleIDAuthorizationScopes.fullName,
          ],
        );

        final appleIdToken = credential.identityToken!;
        if (appleIdToken.isNotEmpty) {
          final deviceInfo = await _getDeviceInfo();
          final params = SocialSigninParams(type: "apple", idToken: appleIdToken);
          final result = await appleSigninUseCase.call(params, deviceInfo);

          result.fold(
            (failure) => emit(SocialAuthFailure(failure.message)),
            (res) {
              _saveSession(res);
              emit(SocialAuthSuccess(res));
            },
          );
        }
      } on SignInWithAppleAuthorizationException catch (e) {
        if (e.code == AuthorizationErrorCode.canceled) {
          emit(SocialAuthInitial());            // ← canceled ≠ failure
        } else {
          emit(SocialAuthFailure("Apple Sign-In Error: ${e.message}"));
        }
      } catch (error) {
        emit(SocialAuthFailure('Sign in failed: $error'));
      }
    });
  }

  // ── Save session to storage
  void _saveSession(SocialSigninEntity res) {
    locator<DioApiService>().setAccessToken(res.token);
    if (res.patient.isProfileCompleted) {
      locator<StorageService>().setToken(res.token);
      locator<StorageService>().setLoginStatus(true);
      locator<StorageService>().setUID(res.patient.id);
      locator<StorageService>().setUserData(res.patient);
      locator<StorageService>().setUsername(res.patient.username ?? "");
    }
  }

  // ── Collect device info for API headers
  Future<Map<String, String>> _getDeviceInfo() async {
    final deviceInfo = DeviceInfoPlugin();
    final packageInfo = await PackageInfo.fromPlatform();
    final uuid = const Uuid();

    String model = 'Unknown';
    String uniqueId = uuid.v4();

    if (Platform.isAndroid) {
      final android = await deviceInfo.androidInfo;
      model = android.model ?? 'Android';
      uniqueId = android.id ?? uuid.v4();
    } else if (Platform.isIOS) {
      final ios = await deviceInfo.iosInfo;
      model = ios.utsname.machine ?? 'iOS';
      uniqueId = ios.identifierForVendor ?? uuid.v4();
    }

    final fcmToken = locator<StorageService>().getFCMToken;

    return {
      'deviceUniqueId': model,
      'deviceModel': uniqueId,
      'fcmToken': fcmToken ?? '',
      'user-agent': '${packageInfo.appName}/${packageInfo.version} (${Platform.operatingSystem})',
    };
  }
}
```

---

## STEP 10 — EVENTS & STATES

```dart
// social_auth_event.dart
part of 'social_auth_bloc.dart';

abstract class SocialAuthEvent extends Equatable {
  const SocialAuthEvent();
  @override List<Object?> get props => [];
}

class SignInWithGoogleRequested extends SocialAuthEvent {}
class SignInWithAppleRequested extends SocialAuthEvent {}

// social_auth_state.dart
part of 'social_auth_bloc.dart';

abstract class SocialAuthState extends Equatable {
  const SocialAuthState();
  @override List<Object?> get props => [];
}

class SocialAuthInitial extends SocialAuthState {}
class SocialAuthLoading extends SocialAuthState {}

class SocialAuthSuccess extends SocialAuthState {
  final SocialSigninEntity socialSignin;
  const SocialAuthSuccess(this.socialSignin);
  @override List<Object?> get props => [socialSignin];
}

class SocialAuthFailure extends SocialAuthState {
  final String message;
  const SocialAuthFailure(this.message);
  @override List<Object?> get props => [message];
}
```

---

## STEP 11 — SOCIAL SIGNIN SCREEN

```dart
class SocialSigninScreen extends StatefulWidget {
  const SocialSigninScreen({super.key});
  @override State<SocialSigninScreen> createState() => _SocialSigninScreenState();
}

class _SocialSigninScreenState extends State<SocialSigninScreen> {
  @override
  Widget build(BuildContext context) {
    return BlocListener<SocialAuthBloc, SocialAuthState>(
      listener: (context, state) async {
        if (state is SocialAuthInitial) {
          context.loaderOverlay.hide();
        } else if (state is SocialAuthLoading) {
          context.loaderOverlay.show();
        } else if (state is SocialAuthSuccess) {
          context.loaderOverlay.hide();
          final user = state.socialSignin;
          context.read<UserBloc>().add(SetUser(user.patient));

          if (!user.patient.isProfileCompleted) {
            if (!user.isUserDetailsFilled) {
              context.goNamed(Routes.createProfile.name, extra: user);
            } else if (!user.isUsernameAdded) {
              context.goNamed(Routes.createUsername.name);
            }
          } else {
            context.goNamed(Routes.home.name);
            context.read<ConversationsBloc>().add(ConnectIfLoggedInEvent());
          }
        } else if (state is SocialAuthFailure) {
          context.loaderOverlay.hide();
          AwesomeNotifier.showSnackBar(
            context: context, title: 'Oops!',
            message: state.message, contentType: ContentType.failure,
          );
        }
      },
      child: Padding(
        padding: EdgeInsets.symmetric(horizontal: 16.5.w, vertical: 16.5.h),
        child: Form(
          child: Column(
            mainAxisAlignment: MainAxisAlignment.end,
            mainAxisSize: MainAxisSize.max,
            children: [
              VerticalSpacing(50),
              // ── Hero logo
              Hero(
                tag: 'appLogo',
                child: SvgPicture.asset(
                  ImageConstants.appLogo, height: 317.w, width: 317.h,
                ),
              ),
              TextComponent(text: 'Sign In', fontsize: 24.sp, fontWeight: FontWeight.w600),
              VerticalSpacing(30),
              // ── Platform-specific social buttons
              if (Platform.isAndroid) ...[
                ButtonComponent(
                  isLoading: false,
                  backgroundColor: AppColors.white,
                  fontColor: AppColors.textPrimary,
                  svgIcon: SvgPicture.asset(ImageConstants.googleIcon),
                  buttonText: "Continue With Google",
                  onPressed: () => context.read<SocialAuthBloc>().add(SignInWithGoogleRequested()),
                ),
                VerticalSpacing(10),
              ],
              if (Platform.isIOS) ...[
                ButtonComponent(
                  isLoading: false,
                  backgroundColor: AppColors.white,
                  fontColor: AppColors.textPrimary,
                  svgIcon: SvgPicture.asset(ImageConstants.appleIcon),
                  buttonText: "Continue With Apple",
                  onPressed: () => context.read<SocialAuthBloc>().add(SignInWithAppleRequested()),
                ),
              ],
              VerticalSpacing(70),
              // ── T&C + Privacy links
              FittedBox(
                fit: BoxFit.scaleDown,
                child: RichText(
                  textAlign: TextAlign.center,
                  text: TextSpan(
                    style: context.bodyMedium?.copyWith(
                      color: AppColors.textPrimary, fontSize: 13.sp,
                      fontWeight: FontWeight.w500,
                      fontFamily: FontFamilyEnum.generalSans.name,
                    ),
                    children: [
                      TextSpan(text: 'I accept the '),
                      TextSpan(
                        text: 'Terms and conditions\n ',
                        style: context.bodyMedium?.copyWith(
                          color: AppColors.primary, fontSize: 13.sp,
                          fontWeight: FontWeight.w500,
                          fontFamily: FontFamilyEnum.generalSans.name,
                        ),
                        recognizer: TapGestureRecognizer()
                          ..onTap = () => context.pushNamed(Routes.termAndConditionScreen.name),
                      ),
                      TextSpan(text: 'and '),
                      TextSpan(
                        text: 'Privacy Policy',
                        style: context.bodyMedium?.copyWith(
                          color: AppColors.primary, fontSize: 13.sp,
                          fontWeight: FontWeight.w500,
                          fontFamily: FontFamilyEnum.generalSans.name,
                        ),
                        recognizer: TapGestureRecognizer()
                          ..onTap = () => context.pushNamed(Routes.privacyScreen.name),
                      ),
                    ],
                  ),
                ),
              ),
              VerticalSpacing(22),
            ],
          ),
        ),
      ),
    );
  }
}
```

---

## STEP 12 — OTP SCREEN (Pinput)

```dart
class VerifyOTPScreen extends StatefulWidget {
  final String maskedEmail;      // e.g. "jo*******@gmail.com"
  final VoidCallback onResend;
  final Function(String otp) onVerify;
  const VerifyOTPScreen({super.key, required this.maskedEmail, required this.onResend, required this.onVerify});
  @override State<VerifyOTPScreen> createState() => _VerifyOTPScreenState();
}

class _VerifyOTPScreenState extends State<VerifyOTPScreen> {
  final TextEditingController otpController = TextEditingController();
  final FocusNode otpFocusNode = FocusNode();

  // ── Pinput decoration
  late final defaultPinDecoration = BoxDecoration(
    color: AppColors.white,
    boxShadow: [BoxShadow(blurRadius: 12, spreadRadius: 0, offset: Offset(0, 4), color: AppColors.secondary.withOpacity(0.04))],
    borderRadius: BorderRadius.circular(12.r),
  );
  late final submittedDecoration = BoxDecoration(
    color: AppColors.primary.withOpacity(0.06),
    borderRadius: BorderRadius.all(Radius.circular(12.r)),
  );
  late final focusedPinDecoration = BoxDecoration(
    color: AppColors.white,
    borderRadius: BorderRadius.circular(12.r),
  );

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(backgroundColor: AppColors.background, elevation: 0, surfaceTintColor: Colors.transparent, toolbarHeight: 40.h),
      body: SingleChildScrollView(
        padding: EdgeInsets.symmetric(horizontal: 16.w, vertical: 24.h),
        child: Column(children: [
          TextComponent(text: 'Verification', fontsize: 24.sp, fontWeight: FontWeight.w600),
          VerticalSpacing(14.h),
          FittedBox(
            fit: BoxFit.scaleDown,
            child: TextComponent(
              textAlign: TextAlign.center,
              text: 'Enter the OTP code sent to ${widget.maskedEmail}',
              fontsize: 14.sp, fontWeight: FontWeight.w400,
            ),
          ),
          VerticalSpacing(28.h),
          Pinput(
            controller: otpController,
            focusNode: otpFocusNode,
            obscureText: true,
            obscuringWidget: Icon(Icons.circle, size: 14.w, color: AppColors.primary),
            length: 5,                        // ← change to user's OTP length
            autofocus: false,
            keyboardType: TextInputType.number,
            onTapOutside: (event) => FocusManager.instance.primaryFocus?.unfocus(),
            defaultPinTheme: PinTheme(height: 56.h, width: 56.w, decoration: defaultPinDecoration),
            submittedPinTheme: PinTheme(height: 56.h, width: 56.w, textStyle: Theme.of(context).textTheme.bodyLarge, decoration: submittedDecoration),
            focusedPinTheme: PinTheme(height: 56.h, width: 56.w, decoration: focusedPinDecoration),
          ),
          VerticalSpacing(28.h),
          // ── Resend link
          FittedBox(
            fit: BoxFit.scaleDown,
            child: RichText(
              text: TextSpan(
                style: context.bodyMedium?.copyWith(color: AppColors.textPrimary, fontSize: 13.sp, fontWeight: FontWeight.w500),
                children: [
                  TextSpan(text: 'Didn\'t receive code? '),
                  TextSpan(
                    text: 'Resend now',
                    style: context.bodyMedium?.copyWith(color: AppColors.primary, fontSize: 13.sp, fontWeight: FontWeight.w500),
                    recognizer: TapGestureRecognizer()..onTap = widget.onResend,
                  ),
                ],
              ),
            ),
          ),
        ]),
      ),
      bottomNavigationBar: Padding(
        padding: EdgeInsets.symmetric(horizontal: 16.w, vertical: 16.h),
        child: ButtonComponent(
          isLoading: false,
          buttonText: "Verify",
          onPressed: () => widget.onVerify(otpController.text),
        ),
      ),
    );
  }
}
```

---

## STEP 13 — CREATE PROFILE SCREEN

```dart
class CreateProfileScreen extends StatefulWidget {
  final SocialSigninEntity user;
  const CreateProfileScreen({super.key, required this.user});
  @override State<CreateProfileScreen> createState() => _CreateProfileScreenState();
}

class _CreateProfileScreenState extends State<CreateProfileScreen> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  // ── Controllers
  late TextEditingController firstNameController;
  late TextEditingController lastNameController;
  late TextEditingController emailController;
  late TextEditingController dateOfBirthController;
  late TextEditingController bioController;

  // ── FocusNodes
  final FocusNode firstNameFocusNode = FocusNode();
  final FocusNode lastNameFocusNode = FocusNode();
  final FocusNode emailFocusNode = FocusNode();
  final FocusNode dateOfBirthFocusNode = FocusNode();
  final FocusNode bioFocusNode = FocusNode();

  @override
  void initState() {
    super.initState();
    // Pre-fill from social data
    final nameParts = widget.user.patient.fullName.trim().split(' ');
    firstNameController = TextEditingController(text: nameParts.isNotEmpty ? nameParts.first : '');
    lastNameController = TextEditingController(text: nameParts.length > 1 ? nameParts.sublist(1).join(' ') : '');
    emailController = TextEditingController(text: widget.user.patient.email);
    dateOfBirthController = TextEditingController(
      text: widget.user.patient.dateOfBirth != null
          ? DateUtilsHelper.formatToDdMmYyyy(widget.user.patient.dateOfBirth!) : '',
    );
    bioController = TextEditingController(text: widget.user.patient.bio ?? '');

    // Pre-load profile pic from network
    Future.microtask(() {
      final profilePic = widget.user.patient.profilePicURL;
      if (profilePic != null && profilePic.isNotEmpty) {
        context.read<MediaCubit>().pickFromNetwork(context, profilePic);
      }
    });
  }

  @override
  void dispose() {
    firstNameController.dispose(); lastNameController.dispose();
    emailController.dispose(); dateOfBirthController.dispose();
    bioController.dispose();
    firstNameFocusNode.dispose(); lastNameFocusNode.dispose();
    emailFocusNode.dispose(); dateOfBirthFocusNode.dispose();
    bioFocusNode.dispose();
    super.dispose();
  }

  void _handleSubmit() {
    if (_formKey.currentState!.validate()) {
      final mediaState = context.read<MediaCubit>().state;
      XFile? imageFile;
      bool removeProfilePic = false;

      if (mediaState is MediaStateSelected && mediaState.networkImageURL == null) {
        imageFile = mediaState.imageFile;
      } else if (mediaState is MediaStateInitial) {
        removeProfilePic = true;
      }

      final params = UserProfileParams(
        fullName: '${firstNameController.text} ${lastNameController.text}',
        email: emailController.text,
        dateOfBirth: DateUtilsHelper.formatDdMmYyyyToIso(dateOfBirthController.text),
        bio: bioController.text,
        profilePic: imageFile,
        removeProfilePic: removeProfilePic,
      );
      context.read<CreateProfileBloc>().add(CreateUserProfile(params));
    }
  }

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      top: false, right: false, left: false, bottom: true,
      child: GradientScaffold(
        appBar: AppBar(forceMaterialTransparency: true),
        body: BlocListener<CreateProfileBloc, CreateProfileState>(
          listener: (context, state) async {
            if (state is Loading) {
              context.loaderOverlay.show();
            } else if (state is CreateProfileSuccess) {
              context.loaderOverlay.hide();
              context.goNamed(Routes.createUsername.name);
            } else if (state is CreateProfileFailure) {
              context.loaderOverlay.hide();
              AwesomeNotifier.showSnackBar(
                context: context, title: 'Oops!',
                message: state.err, contentType: ContentType.failure,
              );
            }
          },
          child: SingleChildScrollView(
            padding: EdgeInsets.symmetric(horizontal: 16.w),
            child: Form(
              key: _formKey,
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  TextComponent(text: 'Add Details', fontsize: 24.sp, fontWeight: FontWeight.w600),
                  VerticalSpacing(25.h),
                  UploadProfileComponent(),
                  VerticalSpacing(38),
                  Row(crossAxisAlignment: CrossAxisAlignment.start, children: [
                    Expanded(child: TextFieldComponent(
                      controller: firstNameController, currentFocus: firstNameFocusNode,
                      nextFocus: lastNameFocusNode, hintText: "Enter first name",
                      keyboardType: TextInputType.name, labelText: "First Name",
                      validator: Validation.validateFirstName,
                    )),
                    HorizontalSpacing(25),
                    Expanded(child: TextFieldComponent(
                      controller: lastNameController, currentFocus: lastNameFocusNode,
                      nextFocus: emailFocusNode, hintText: "Enter last name",
                      keyboardType: TextInputType.name, labelText: "Last Name",
                      validator: Validation.validateLastName,
                    )),
                  ]),
                  VerticalSpacing(16),
                  TextFieldComponent(
                    readOnly: true,                    // ← email is pre-filled, read only
                    maxLength: 254,
                    controller: emailController, currentFocus: emailFocusNode,
                    nextFocus: dateOfBirthFocusNode, hintText: "Enter email here",
                    keyboardType: TextInputType.emailAddress, labelText: "Email Address",
                  ),
                  VerticalSpacing(16),
                  TextFieldComponent(
                    controller: dateOfBirthController, currentFocus: dateOfBirthFocusNode,
                    nextFocus: bioFocusNode, hintText: "dd/mm/yyyy", labelText: "Date of birth",
                    readOnly: true,                    // ← opens calendar dialog
                    suffixIcon: SvgPicture.asset(ImageConstants.calenderIcon, width: 18.w, height: 18.h, fit: BoxFit.scaleDown),
                    onTap: () => DialogUtils.showCalendarDialog(context, onDateSelected: (value) {
                      dateOfBirthController.text = DateUtilsHelper.formatToDdMmYyyy(value);
                    }),
                    validator: Validation.validateDateOfBirth,
                  ),
                  VerticalSpacing(16),
                  TextFieldComponent(
                    borderRadius: 15.r,               // ← bio gets square corners (not pill)
                    controller: bioController, currentFocus: bioFocusNode,
                    hintText: "Enter bio", keyboardType: TextInputType.multiline,
                    textInputAction: TextInputAction.newline,
                    labelText: "Bio (optional)", maxLength: 150,
                    maxLines: 5, minLines: 3, obscureText: false,
                    enableSuggestions: true, autocorrect: true,
                    textCapitalization: TextCapitalization.sentences,
                  ),
                  VerticalSpacing(16),
                ],
              ),
            ),
          ),
        ),
        bottomNavigationBar: Padding(
          padding: EdgeInsets.only(left: 16.w, right: 16.w, bottom: 10.h, top: 10.h),
          child: ButtonComponent(
            fontWeight: FontWeight.w600, isLoading: false,
            buttonText: "Next", onPressed: _handleSubmit,
          ),
        ),
      ),
    );
  }
}
```

---

## STEP 14 — CREATE USERNAME SCREEN

```dart
class CreateUsernameScreen extends StatefulWidget {
  const CreateUsernameScreen({super.key});
  @override State<CreateUsernameScreen> createState() => _CreateUsernameScreenState();
}

class _CreateUsernameScreenState extends State<CreateUsernameScreen> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();
  final TextEditingController usernameController = TextEditingController();
  final FocusNode usernameFocusNode = FocusNode();

  void _handleSubmit() {
    if (_formKey.currentState!.validate()) {
      context.read<CreateUsernameBloc>().add(
        CreateUsername(UsernameParams(username: usernameController.text)),
      );
    }
  }

  @override
  Widget build(BuildContext context) {
    return GradientScaffold(
      appBar: AppBar(forceMaterialTransparency: true),
      body: BlocListener<CreateUsernameBloc, CreateUsernameState>(
        listener: (context, state) async {
          if (state is Loading) {
            context.loaderOverlay.show();
          } else if (state is CreateUsernameSuccess && state.user.patient.isProfileCompleted) {
            context.loaderOverlay.hide();
            context.read<UserBloc>().add(SetUser(state.user.patient));
            context.read<ConversationsBloc>().add(ConnectIfLoggedInEvent());
            context.pushNamed(Routes.success.name, extra: SuccessType.accountCreated);
          } else if (state is CreateUsernameFailure) {
            context.loaderOverlay.hide();
            AwesomeNotifier.showSnackBar(
              context: context, title: 'Oops!',
              message: state.err, contentType: ContentType.failure,
            );
          }
        },
        child: SingleChildScrollView(
          padding: EdgeInsets.symmetric(horizontal: 16.w),
          child: Form(
            key: _formKey,
            child: Column(children: [
              TextComponent(text: 'Create User Name', fontsize: 24.sp, fontWeight: FontWeight.w600),
              VerticalSpacing(14.h),
              FittedBox(fit: BoxFit.scaleDown, child: TextComponent(
                textAlign: TextAlign.center, text: 'Please create your user name',
                fontsize: 14.sp, fontWeight: FontWeight.w400,
              )),
              VerticalSpacing(35.h),
              TextFieldComponent(
                controller: usernameController, currentFocus: usernameFocusNode,
                hintText: "Create Username", keyboardType: TextInputType.name,
                validator: Validation.validateUsername,
              ),
            ]),
          ),
        ),
      ),
      bottomNavigationBar: Padding(
        padding: EdgeInsets.only(left: 16.w, right: 16.w, bottom: 50.h),
        child: ButtonComponent(
          fontWeight: FontWeight.w600, isLoading: false,
          buttonText: "Next", onPressed: _handleSubmit,
        ),
      ),
    );
  }
}
```

---

## STEP 15 — TEXTFIELD COMPONENT USAGE RULES

```dart
// ── Basic field
TextFieldComponent(
  controller: myController,
  currentFocus: myFocusNode,
  nextFocus: nextFocusNode,           // auto-advances on submit
  hintText: "Placeholder",
  labelText: "Field Label",           // shown above field
  keyboardType: TextInputType.text,
  validator: Validation.validateCommon,
)

// ── Password field
TextFieldComponent(
  controller: passwordController,
  currentFocus: passwordFocusNode,
  hintText: "Enter password",
  labelText: "Password",
  obscureText: true,
  suffixIcon: IconButton(icon: Icon(Icons.visibility), onPressed: toggleVisibility),
)

// ── Multi-line (bio, description)
TextFieldComponent(
  borderRadius: 15.r,               // ← override pill shape for multi-line
  controller: bioController,
  currentFocus: bioFocusNode,
  hintText: "Enter bio",
  keyboardType: TextInputType.multiline,
  textInputAction: TextInputAction.newline,
  maxLength: 150, maxLines: 5, minLines: 3,
  textCapitalization: TextCapitalization.sentences,
)

// ── Read-only with action
TextFieldComponent(
  controller: dobController,
  currentFocus: dobFocusNode,
  hintText: "dd/mm/yyyy",
  readOnly: true,                   // ← prevents keyboard
  onTap: () => DialogUtils.showCalendarDialog(context, onDateSelected: ...),
  suffixIcon: SvgPicture.asset(ImageConstants.calenderIcon),
)
```

---

## STEP 16 — BUTTON COMPONENT USAGE RULES

```dart
// ── Primary gradient button (default)
ButtonComponent(isLoading: false, buttonText: "Submit", onPressed: _handleSubmit)

// ── White solid button (social auth)
ButtonComponent(
  isLoading: false,
  backgroundColor: AppColors.white,
  fontColor: AppColors.textPrimary,
  svgIcon: SvgPicture.asset(ImageConstants.googleIcon),
  buttonText: "Continue With Google",
  onPressed: () { ... },
)

// ── Loading state
ButtonComponent(isLoading: state is Loading, buttonText: "Submit", onPressed: _submit)

// ── Outline button
ButtonComponent(
  backgroundColor: Colors.transparent,
  borderColor: AppColors.primary,
  fontColor: AppColors.primary,
  buttonText: "Cancel", onPressed: () {},
)
```

---

## STEP 17 — LOADING PATTERN (loaderOverlay — NOT Loader.show())

Auth screens use `loader_overlay` package, NOT the global Loader system.

```dart
// Show
context.loaderOverlay.show();

// Hide
context.loaderOverlay.hide();

// Pattern in BlocListener
listener: (context, state) async {
  if (state is SocialAuthLoading) {
    context.loaderOverlay.show();
  } else if (state is SocialAuthSuccess || state is SocialAuthFailure || state is SocialAuthInitial) {
    context.loaderOverlay.hide();
    // handle result...
  }
},
```

---

## STEP 18 — DI REGISTRATION (service_locator.dart)

```dart
// DataSource
locator.registerLazySingleton<AuthDatasource>(() => AuthDatasourceImpl());

// Repository (same impl for both interfaces)
locator.registerLazySingleton<GoogleSigninRepository>(
  () => AuthRepositoryImpl(locator<AuthDatasource>()),
);
locator.registerLazySingleton<AppleSigninRepository>(
  () => AuthRepositoryImpl(locator<AuthDatasource>()),
);

// UseCases
locator.registerLazySingleton(
  () => GoogleSigninUseCase(locator<GoogleSigninRepository>()),
);
locator.registerLazySingleton(
  () => AppleSigninUseCase(locator<AppleSigninRepository>()),
);

// Bloc — registerFactory (fresh per navigation)
locator.registerFactory(
  () => SocialAuthBloc(locator<GoogleSigninUseCase>(), locator<AppleSigninUseCase>()),
);
```

---

## STEP 19 — ROUTES ENUM ENTRIES

```dart
enum Routes {
  socialSignin('/auth/social-signin'),
  createProfile('/profile/create-profile'),
  createUsername('/profile/create-username'),
  success('/success'),
  termAndConditionScreen('/setting/term_and_condition_screen'),
  privacyScreen('/setting/privacy_screen'),
  // ...
}
```

---

## STEP 20 — GOROUTER REGISTRATION

```dart
// Social signin route (no BLoC needed at route level — injected in screen)
GoRoute(
  path: Routes.socialSignin.path,
  name: Routes.socialSignin.name,
  pageBuilder: (context, state) => AppRouter._buildPage(
    BlocProvider(
      create: (_) => locator<SocialAuthBloc>(),
      child: const SocialSigninScreen(),
    ),
    state,
  ),
),

// Create profile (receives SocialSigninEntity as extra)
GoRoute(
  path: Routes.createProfile.path,
  name: Routes.createProfile.name,
  pageBuilder: (context, state) {
    final user = state.extra as SocialSigninEntity;
    return AppRouter._buildPage(
      MultiBlocProvider(
        providers: [
          BlocProvider(create: (_) => locator<CreateProfileBloc>()),
          BlocProvider(create: (_) => locator<MediaCubit>()),
        ],
        child: CreateProfileScreen(user: user),
      ),
      state,
    );
  },
),

// Create username
GoRoute(
  path: Routes.createUsername.path,
  name: Routes.createUsername.name,
  pageBuilder: (context, state) => AppRouter._buildPage(
    BlocProvider(
      create: (_) => locator<CreateUsernameBloc>(),
      child: const CreateUsernameScreen(),
    ),
    state,
  ),
),
```

---

## STEP 21 — DEPENDENCIES CHECKLIST

```yaml
dependencies:
  flutter_bloc: ^8.x
  bloc: ^8.x
  dartz: ^0.10.x
  get_it: ^7.x
  go_router: ^13.x
  flutter_screenutil: ^5.x
  pinput: ^3.x                   # OTP input
  google_sign_in: ^6.x           # Google auth
  sign_in_with_apple: ^6.x       # Apple auth
  device_info_plus: ^9.x         # Device info headers
  package_info_plus: ^4.x        # App version in headers
  uuid: ^4.x                     # Fallback unique ID
  loader_overlay: ^3.x           # Auth screen loading
  awesome_snackbar_content: ^0.1.x
  intl: ^0.19.x
  equatable: ^2.x
  flutter_svg: ^2.x
  gradient_borders: ^1.x         # TextFieldComponent focused border
  image_picker: ^1.x             # Profile pic upload
  skeletonizer: ^1.x
```

---

## VALIDATION CHECKLIST

- [ ] All `Validation.*` validators in `lib/core/utils/validations.dart`
- [ ] `DateUtilsHelper` in `lib/core/utils/date_utils.dart`
- [ ] `DialogUtils.showCalendarDialog` in `lib/core/utils/dialog_utils.dart`
- [ ] `SocialSignin` model extends `SocialSigninEntity` — no toEntity() needed
- [ ] BLoC `part of` pattern — events/states use `part of` directive
- [ ] `_googleSignIn.signOut()` called BEFORE `signIn()` (force account picker)
- [ ] Apple canceled: `AuthorizationErrorCode.canceled` → `SocialAuthInitial()` (not failure)
- [ ] Device info collected before every API call
- [ ] `RequestConfig(headers: deviceInfo)` passed to datasource
- [ ] `_saveSession()` only saves to StorageService if `isProfileCompleted == true`
- [ ] Loading: `context.loaderOverlay.show/hide()` — NOT `Loader.show/hide()`
- [ ] Error: `AwesomeNotifier.showSnackBar(contentType: ContentType.failure)`
- [ ] Navigation: `context.goNamed(Routes.xxx.name)` — named routes
- [ ] Profile screen: `extra: user` passed as GoRouter extra
- [ ] Date field: readOnly + onTap → DialogUtils.showCalendarDialog
- [ ] Date conversion: display `dd/MM/yyyy` → API `ISO8601` via `formatDdMmYyyyToIso`
- [ ] Bio field: borderRadius 15.r (not pill), maxLines 5, minLines 3
- [ ] Email field: readOnly (pre-filled from social data)
- [ ] Form validation: `_formKey.currentState!.validate()`
- [ ] All controllers + focusNodes disposed in dispose()
- [ ] DI: same `AuthRepositoryImpl` for both Google and Apple repos
- [ ] Router: profile screen uses `state.extra as SocialSigninEntity`

**Why:** Extracted from production cier_check_user app. This is the exact architecture, UX patterns, validators, formatters, and navigation flow used in the live auth feature.
**How to apply:** Ask user ONLY for API details and auth method. Generate all 21 steps automatically from this skill.
