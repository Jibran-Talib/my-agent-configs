---
name: Flutter User Profile Skill
description: Complete reusable skill for Flutter Clean Architecture + BLoC projects — generates full user profile feature (view, edit, avatar upload) following production patterns from cier_check_user
type: project
---

# FLUTTER USER PROFILE SKILL (Global)

## WHEN TO APPLY
Any Flutter feature that needs: view profile, edit profile fields, upload/remove avatar photo.

---

## FOLDER STRUCTURE

```
lib/
├── shared/
│   ├── entities/
│   │   └── user_entity.dart              ← domain entity (immutable, copyWith)
│   └── models/
│       └── user_model.dart               ← extends entity, fromJson/toJson/empty()
├── core/
│   └── params/
│       └── user_profile_params.dart      ← FormData params with XFile support
└── features/
    └── user/
        ├── data/
        │   ├── datasources/
        │   │   └── user_data_source.dart  ← abstract + impl
        │   └── repositories/
        │       └── user_repository_impl.dart
        ├── domain/
        │   ├── repositories/
        │   │   ├── get_profile_repository.dart
        │   │   └── edit_profile_repository.dart
        │   └── usecases/
        │       ├── get_profile_usecase.dart
        │       └── edit_profile_usecase.dart
        └── presentation/
            ├── bloc/
            │   └── user_profile/
            │       ├── user_profile_bloc.dart   ← part files
            │       ├── user_profile_event.dart
            │       └── user_profile_state.dart
            └── screens/
                └── edit_profile_screen.dart
```

---

## NAMING CONVENTIONS

| Layer | Pattern | Example |
|---|---|---|
| Entity | `FeatureEntity` | `UserEntity` |
| Model | `FeatureModel` or plain name | `User` |
| Params | `FeatureParams` | `UserProfileParams` |
| DataSource abstract | `FeatureDataSource` | `UserDataSource` |
| DataSource impl | `FeatureDataSourceImpl` | `UserDataSourceImpl` |
| Aggregate repo interface | `FeatureRepositoryInternal` | `UserRepositoryInternal` |
| Repository impl | `FeatureRepositoryImpl` | `UserRepositoryImpl` |
| Repository interfaces | `VerbFeatureRepository` | `EditProfileRepository` |
| UseCase | `VerbFeatureUseCase` | `EditProfileUseCase` |
| Bloc | `FeatureBloc` | `UserProfileBloc` |
| Event base | `FeatureEvent` | `UserProfileEvent` |
| State base | `FeatureState` | `UserProfileState` |
| Screen | `VerbFeatureScreen` | `EditProfileScreen` |

---

## ENTITY

```dart
// lib/shared/entities/user_entity.dart
class UserEntity {
  final String id;
  final String fullName;
  final String? username;
  final String email;
  final String? profilePicURL;
  final String? bio;
  final DateTime? dateOfBirth;
  final bool isEmailVerified;
  final bool isPhoneNumberVerified;
  final bool isNotification;
  final bool isProfileCompleted;
  final bool isPrivate;
  final int followingCount;
  final int followersCount;
  final int postLength;
  final bool isActive;
  final bool isDeleted;
  final DateTime createdAt;
  final DateTime updatedAt;

  const UserEntity({
    required this.id,
    required this.fullName,
    this.username,
    required this.email,
    this.profilePicURL,
    this.bio,
    this.dateOfBirth,
    required this.isEmailVerified,
    required this.isPhoneNumberVerified,
    required this.isNotification,
    required this.isProfileCompleted,
    required this.isPrivate,
    required this.followingCount,
    required this.followersCount,
    required this.postLength,
    required this.isActive,
    required this.isDeleted,
    required this.createdAt,
    required this.updatedAt,
  });

  UserEntity copyWith({
    String? id,
    String? fullName,
    String? username,
    String? email,
    String? profilePicURL,
    String? bio,
    DateTime? dateOfBirth,
    bool? isEmailVerified,
    bool? isPhoneNumberVerified,
    bool? isNotification,
    bool? isProfileCompleted,
    bool? isPrivate,
    int? followingCount,
    int? followersCount,
    int? postLength,
    bool? isActive,
    bool? isDeleted,
    DateTime? createdAt,
    DateTime? updatedAt,
  }) {
    return UserEntity(
      id: id ?? this.id,
      fullName: fullName ?? this.fullName,
      username: username ?? this.username,
      email: email ?? this.email,
      profilePicURL: profilePicURL ?? this.profilePicURL,
      bio: bio ?? this.bio,
      dateOfBirth: dateOfBirth ?? this.dateOfBirth,
      isEmailVerified: isEmailVerified ?? this.isEmailVerified,
      isPhoneNumberVerified: isPhoneNumberVerified ?? this.isPhoneNumberVerified,
      isNotification: isNotification ?? this.isNotification,
      isProfileCompleted: isProfileCompleted ?? this.isProfileCompleted,
      isPrivate: isPrivate ?? this.isPrivate,
      followingCount: followingCount ?? this.followingCount,
      followersCount: followersCount ?? this.followersCount,
      postLength: postLength ?? this.postLength,
      isActive: isActive ?? this.isActive,
      isDeleted: isDeleted ?? this.isDeleted,
      createdAt: createdAt ?? this.createdAt,
      updatedAt: updatedAt ?? this.updatedAt,
    );
  }
}
```

> **Adapt fields to project:** keep only fields your API returns. Minimum required: `id`, `fullName`, `email`, `profilePicURL`, `username`, `bio`, `createdAt`, `updatedAt`.

---

## MODEL

```dart
// lib/shared/models/user_model.dart
import 'dart:convert';
import 'package:your_app/shared/entities/user_entity.dart';

List<User> userFromJson(List list) =>
    List<User>.from(list.map((x) => User.fromJson(x)));

String userToJson(List<User> data) =>
    json.encode(List<dynamic>.from(data.map((x) => x.toJson())));

class User extends UserEntity {
  const User({
    required super.id,
    required super.fullName,
    super.username,
    required super.email,
    super.profilePicURL = '',
    super.bio = '',
    super.dateOfBirth,
    required super.isEmailVerified,
    required super.isPhoneNumberVerified,
    required super.isNotification,
    required super.isProfileCompleted,
    required super.isPrivate,
    required super.followingCount,
    required super.followersCount,
    required super.postLength,
    required super.isActive,
    required super.isDeleted,
    required super.createdAt,
    required super.updatedAt,
  });

  factory User.fromJson(Map<String, dynamic> json) {
    final dob = json['dateOfBirth'];
    return User(
      id: json['_id'] ?? '',
      fullName: json['fullName'] ?? '',
      username: (json['username']?.toString().isEmpty ?? true)
          ? null
          : json['username'],
      email: json['email'] ?? '',
      profilePicURL: json['profilePicURL'] ?? '',
      bio: (json['bio']?.toString().isEmpty ?? true) ? null : json['bio'],
      dateOfBirth: (dob == null || dob.toString().isEmpty)
          ? null
          : DateTime.tryParse(dob.toString()),
      isEmailVerified: json['isEmailVerified'] ?? false,
      isPhoneNumberVerified: json['isPhoneNumberVerified'] ?? false,
      isNotification: json['isNotification'] ?? true,
      isProfileCompleted: json['isProfileCompleted'] ?? false,
      isPrivate: json['isPrivate'] ?? false,
      followingCount: json['followingCount'] ?? 0,
      followersCount: json['followersCount'] ?? 0,
      postLength: json['postLength'] ?? 0,
      isActive: json['isActive'] ?? false,
      isDeleted: json['isDeleted'] ?? false,
      createdAt: DateTime.tryParse(json['createdAt'] ?? '') ?? DateTime.now(),
      updatedAt: DateTime.tryParse(json['updatedAt'] ?? '') ?? DateTime.now(),
    );
  }

  Map<String, dynamic> toJson() => {
    '_id': id,
    'fullName': fullName,
    'username': username,
    'email': email,
    'profilePicURL': profilePicURL,
    'bio': bio,
    'dateOfBirth': dateOfBirth?.toIso8601String(),
    'isEmailVerified': isEmailVerified,
    'isPhoneNumberVerified': isPhoneNumberVerified,
    'isNotification': isNotification,
    'isProfileCompleted': isProfileCompleted,
    'isPrivate': isPrivate,
    'followingCount': followingCount,
    'followersCount': followersCount,
    'postLength': postLength,
    'isActive': isActive,
    'isDeleted': isDeleted,
    'createdAt': createdAt.toIso8601String(),
    'updatedAt': updatedAt.toIso8601String(),
  };

  factory User.fromEntity(UserEntity entity) => User(
    id: entity.id,
    fullName: entity.fullName,
    username: entity.username,
    email: entity.email,
    profilePicURL: entity.profilePicURL,
    bio: entity.bio,
    dateOfBirth: entity.dateOfBirth,
    isEmailVerified: entity.isEmailVerified,
    isPhoneNumberVerified: entity.isPhoneNumberVerified,
    isNotification: entity.isNotification,
    isProfileCompleted: entity.isProfileCompleted,
    isPrivate: entity.isPrivate,
    followersCount: entity.followersCount,
    followingCount: entity.followingCount,
    postLength: entity.postLength,
    isActive: entity.isActive,
    isDeleted: entity.isDeleted,
    createdAt: entity.createdAt,
    updatedAt: entity.updatedAt,
  );

  factory User.empty() => User(
    id: '',
    fullName: '',
    username: null,
    email: '',
    profilePicURL: '',
    bio: null,
    dateOfBirth: null,
    isEmailVerified: false,
    isPhoneNumberVerified: false,
    isProfileCompleted: false,
    isPrivate: false,
    followingCount: 0,
    followersCount: 0,
    isNotification: false,
    postLength: 0,
    isActive: false,
    isDeleted: false,
    createdAt: DateTime.fromMillisecondsSinceEpoch(0),
    updatedAt: DateTime.fromMillisecondsSinceEpoch(0),
  );
}
```

---

## PARAMS (with FormData / file upload)

```dart
// lib/core/params/user_profile_params.dart
import 'package:dio/dio.dart' as dio;
import 'package:http_parser/http_parser.dart';
import 'package:image_picker/image_picker.dart';
import 'package:mime/mime.dart';

class UserProfileParams {
  final String fullName;
  final String dateOfBirth; // ISO 8601 string
  final String email;
  final String? username;
  final XFile? profilePic;  // null = keep existing
  final String bio;
  final bool removeProfilePic; // true = remove avatar

  UserProfileParams({
    required this.fullName,
    required this.dateOfBirth,
    required this.email,
    this.username,
    required this.profilePic,
    required this.bio,
    this.removeProfilePic = false,
  });

  Future<dio.FormData> toFormData() async {
    dio.MultipartFile? imageMultipartFile;

    if (profilePic != null) {
      final mimeType = lookupMimeType(profilePic!.path);
      final mimeTypeParts = mimeType!.split('/');
      imageMultipartFile = await dio.MultipartFile.fromFile(
        profilePic!.path,
        filename: profilePic!.path.split('/').last,
        contentType: MediaType(mimeTypeParts[0], mimeTypeParts[1]),
      );
    }

    return dio.FormData.fromMap({
      'fullName': fullName,
      'dateOfBirth': dateOfBirth,
      'email': email,
      if (username != null) 'username': username,
      if (imageMultipartFile != null) 'profilePic': imageMultipartFile,
      'bio': bio,
      'removeProfilePic': removeProfilePic,
    });
  }
}
```

> **Adapt:** Remove `removeProfilePic` if API doesn't support it. Rename `profilePic` to match your API field name.

---

## DOMAIN REPOSITORIES

```dart
// lib/features/user/domain/repositories/get_profile_repository.dart
import 'package:dartz/dartz.dart';
import 'package:your_app/core/errors/failure.dart';
import 'package:your_app/shared/entities/user_entity.dart';

abstract class GetProfileRepository {
  Future<Either<Failure, UserEntity>> getProfile();
}
```

```dart
// lib/features/user/domain/repositories/edit_profile_repository.dart
import 'package:dartz/dartz.dart';
import 'package:your_app/core/errors/failure.dart';
import 'package:your_app/core/params/user_profile_params.dart';
import 'package:your_app/shared/entities/user_entity.dart';

abstract class EditProfileRepository {
  Future<Either<Failure, UserEntity>> editProfile({
    required UserProfileParams params,
  });
}
```

---

## USE CASES

```dart
// lib/features/user/domain/usecases/get_profile_usecase.dart
import 'package:dartz/dartz.dart';
import 'package:your_app/core/errors/failure.dart';
import 'package:your_app/shared/entities/user_entity.dart';
import 'package:your_app/features/user/domain/repositories/get_profile_repository.dart';

class GetProfileUseCase {
  final GetProfileRepository repository;
  GetProfileUseCase(this.repository);

  Future<Either<Failure, UserEntity>> call() async {
    return await repository.getProfile();
  }
}
```

```dart
// lib/features/user/domain/usecases/edit_profile_usecase.dart
import 'package:dartz/dartz.dart';
import 'package:your_app/core/errors/failure.dart';
import 'package:your_app/core/params/user_profile_params.dart';
import 'package:your_app/shared/entities/user_entity.dart';
import 'package:your_app/features/user/domain/repositories/edit_profile_repository.dart';

class EditProfileUseCase {
  final EditProfileRepository repository;
  EditProfileUseCase(this.repository);

  Future<Either<Failure, UserEntity>> call(UserProfileParams params) async {
    return await repository.editProfile(params: params);
  }
}
```

---

## DATA SOURCE

```dart
// lib/features/user/data/datasources/user_data_source.dart
import 'package:dartz/dartz.dart';
import 'package:flutter/foundation.dart';
import 'package:your_app/core/constants/api_endpoints.dart';
import 'package:your_app/core/errors/exceptions.dart';
import 'package:your_app/core/errors/failure.dart';
import 'package:your_app/core/network/api_response.dart';
import 'package:your_app/core/network/dio_api_service.dart';
import 'package:your_app/core/params/user_profile_params.dart';
import 'package:your_app/core/services/service_locator.dart';
import 'package:your_app/shared/models/user_model.dart';

abstract class UserDataSource {
  Future<Either<Failure, User>> getProfile();
  Future<Either<Failure, User>> editProfile(UserProfileParams params);
}

class UserDataSourceImpl implements UserDataSource {
  // RULE: always use locator — never constructor-inject DioApiService
  final DioApiService apiService = locator<DioApiService>();

  @override
  Future<Either<Failure, User>> getProfile() async {
    try {
      ApiResponse res = await apiService.get(ApiEndpoints.getProfile);
      User userData = User.fromJson(res.data);
      return Right(userData);
    } on ApiException catch (e) {
      debugPrint('getProfile error: $e');
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      debugPrint('getProfile unexpected error: $e');
      return Left(ServerFailure(message: 'Unexpected error: ${e.toString()}'));
    }
  }

  @override
  Future<Either<Failure, User>> editProfile(UserProfileParams params) async {
    try {
      // Use upload() for multipart/form-data (avatar + fields)
      ApiResponse res = await apiService.upload(
        ApiEndpoints.editProfile,
        formData: await params.toFormData(),
      );
      User userData = User.fromJson(res.data);
      return Right(userData);
    } on ApiException catch (e) {
      debugPrint('editProfile error: $e');
      return Left(ServerFailure(message: e.message));
    } catch (e) {
      debugPrint('editProfile unexpected error: $e');
      return Left(ServerFailure(message: 'Unexpected error: ${e.toString()}'));
    }
  }
}
```

---

## REPOSITORY IMPLEMENTATION

```dart
// lib/features/user/data/repositories/user_repository_impl.dart
import 'package:dartz/dartz.dart';
import 'package:your_app/core/errors/failure.dart';
import 'package:your_app/core/params/user_profile_params.dart';
import 'package:your_app/features/user/data/datasources/user_data_source.dart';
import 'package:your_app/features/user/domain/repositories/edit_profile_repository.dart';
import 'package:your_app/features/user/domain/repositories/get_profile_repository.dart';
import 'package:your_app/shared/entities/user_entity.dart';

// Aggregate interface — lists all repository interfaces this impl satisfies
abstract class UserRepositoryInternal
    implements GetProfileRepository, EditProfileRepository {}

class UserRepositoryImpl implements UserRepositoryInternal {
  final UserDataSource dataSource;
  const UserRepositoryImpl(this.dataSource);

  @override
  Future<Either<Failure, UserEntity>> getProfile() async {
    final response = await dataSource.getProfile();
    return response.fold(
      (left) => Left(ServerFailure(message: left.message)),
      (right) => Right(right),
    );
  }

  @override
  Future<Either<Failure, UserEntity>> editProfile({
    required UserProfileParams params,
  }) async {
    final response = await dataSource.editProfile(params);
    return response.fold(
      (left) => Left(ServerFailure(message: left.message)),
      (right) => Right(right),
    );
  }
}
```

---

## BLOC — EVENT FILE

```dart
// lib/features/user/presentation/bloc/user_profile/user_profile_event.dart
part of 'user_profile_bloc.dart';

sealed class UserProfileEvent extends Equatable {
  const UserProfileEvent();

  @override
  List<Object?> get props => [];
}

final class GetUserProfileEvent extends UserProfileEvent {
  const GetUserProfileEvent();
}

final class EditUserProfileEvent extends UserProfileEvent {
  final UserProfileParams params;
  const EditUserProfileEvent(this.params);

  @override
  List<Object> get props => [params];
}
```

---

## BLOC — STATE FILE

```dart
// lib/features/user/presentation/bloc/user_profile/user_profile_state.dart
part of 'user_profile_bloc.dart';

sealed class UserProfileState extends Equatable {
  const UserProfileState();

  @override
  List<Object?> get props => [];
}

final class UserProfileInitial extends UserProfileState {}

final class UserProfileLoading extends UserProfileState {}

final class UserProfileLoaded extends UserProfileState {
  final UserEntity user;
  const UserProfileLoaded(this.user);

  @override
  List<Object?> get props => [user];
}

final class UserProfileEditSuccess extends UserProfileState {
  final UserEntity user;
  const UserProfileEditSuccess(this.user);

  @override
  List<Object?> get props => [user];
}

final class UserProfileFailure extends UserProfileState {
  final String err;
  const UserProfileFailure(this.err);

  @override
  List<Object?> get props => [err];
}
```

---

## BLOC — BLOC FILE

```dart
// lib/features/user/presentation/bloc/user_profile/user_profile_bloc.dart
import 'package:bloc/bloc.dart';
import 'package:equatable/equatable.dart';
import 'package:your_app/core/params/user_profile_params.dart';
import 'package:your_app/core/services/service_locator.dart';
import 'package:your_app/core/services/storage_service.dart';
import 'package:your_app/features/user/domain/usecases/edit_profile_usecase.dart';
import 'package:your_app/features/user/domain/usecases/get_profile_usecase.dart';
import 'package:your_app/shared/entities/user_entity.dart';

part 'user_profile_event.dart';
part 'user_profile_state.dart';

class UserProfileBloc extends Bloc<UserProfileEvent, UserProfileState> {
  final GetProfileUseCase getProfileUseCase;
  final EditProfileUseCase editProfileUseCase;

  UserProfileBloc(this.getProfileUseCase, this.editProfileUseCase)
      : super(UserProfileInitial()) {
    on<GetUserProfileEvent>(_getProfile);
    on<EditUserProfileEvent>(_editProfile);
  }

  Future<void> _getProfile(
    GetUserProfileEvent event,
    Emitter<UserProfileState> emit,
  ) async {
    emit(UserProfileLoading());
    final result = await getProfileUseCase.call();
    result.fold(
      (failure) => emit(UserProfileFailure(failure.message)),
      (user) => emit(UserProfileLoaded(user)),
    );
  }

  Future<void> _editProfile(
    EditUserProfileEvent event,
    Emitter<UserProfileState> emit,
  ) async {
    emit(UserProfileLoading());
    final result = await editProfileUseCase.call(event.params);
    result.fold(
      (failure) => emit(UserProfileFailure(failure.message)),
      (user) {
        // Persist updated user to local storage
        locator<StorageService>().setUserData(user);
        emit(UserProfileEditSuccess(user));
      },
    );
  }
}
```

---

## SCREEN — EDIT PROFILE

```dart
// lib/features/user/presentation/screens/edit_profile_screen.dart
import 'package:awesome_snackbar_content/awesome_snackbar_content.dart';
import 'package:flutter/material.dart';
import 'package:flutter_bloc/flutter_bloc.dart';
import 'package:flutter_screenutil/flutter_screenutil.dart';
import 'package:go_router/go_router.dart';
import 'package:image_picker/image_picker.dart';
import 'package:loader_overlay/loader_overlay.dart';

import 'package:your_app/core/constants/app_colors.dart';
import 'package:your_app/core/params/user_profile_params.dart';
import 'package:your_app/core/utils/awesome_notifier.dart';
import 'package:your_app/core/utils/date_utils.dart';
import 'package:your_app/core/utils/dialog_utils.dart';
import 'package:your_app/core/utils/spacing.dart';
import 'package:your_app/core/utils/validations.dart';
import 'package:your_app/features/general/media_cubit.dart';
import 'package:your_app/features/user/presentation/bloc/user_profile/user_profile_bloc.dart';
import 'package:your_app/shared/bloc/user_bloc.dart'; // global user bloc
import 'package:your_app/shared/widgets/app_bar_component.dart';
import 'package:your_app/shared/widgets/button_component.dart';
import 'package:your_app/shared/widgets/gradient_scaffold.dart';
import 'package:your_app/shared/widgets/text_field_component.dart';
import 'package:your_app/shared/widgets/upload_profile_component.dart';

class EditProfileScreen extends StatefulWidget {
  const EditProfileScreen({Key? key}) : super(key: key);

  @override
  State<EditProfileScreen> createState() => _EditProfileScreenState();
}

class _EditProfileScreenState extends State<EditProfileScreen> {
  final GlobalKey<FormState> _formKey = GlobalKey<FormState>();

  late TextEditingController _firstNameController;
  late TextEditingController _lastNameController;
  late TextEditingController _emailController;
  late TextEditingController _usernameController;
  late TextEditingController _dateOfBirthController;
  late TextEditingController _bioController;

  final _firstNameFocus = FocusNode();
  final _lastNameFocus = FocusNode();
  final _emailFocus = FocusNode();
  final _usernameFocus = FocusNode();
  final _dobFocus = FocusNode();
  final _bioFocus = FocusNode();

  @override
  void initState() {
    super.initState();
    _initControllers();
  }

  void _initControllers() {
    // Read current user from global UserBloc and pre-fill form
    final state = context.read<UserBloc>().state;
    if (state is LoggedUser) {
      final user = state.userData;

      final nameParts = user.fullName.trim().split(' ');
      _firstNameController = TextEditingController(
        text: nameParts.isNotEmpty ? nameParts.first : '',
      );
      _lastNameController = TextEditingController(
        text: nameParts.length > 1 ? nameParts.sublist(1).join(' ') : '',
      );
      _emailController = TextEditingController(text: user.email);
      _usernameController = TextEditingController(text: user.username ?? '');
      _dateOfBirthController = TextEditingController(
        text: user.dateOfBirth != null
            ? DateUtilsHelper.formatToDdMmYyyy(user.dateOfBirth!)
            : '',
      );
      _bioController = TextEditingController(text: user.bio ?? '');

      // Load existing avatar into MediaCubit for display
      WidgetsBinding.instance.addPostFrameCallback((_) {
        final picUrl = user.profilePicURL;
        if (picUrl != null && picUrl.isNotEmpty) {
          context.read<MediaCubit>().pickFromNetwork(context, picUrl);
        }
      });
    } else {
      _firstNameController = TextEditingController();
      _lastNameController = TextEditingController();
      _emailController = TextEditingController();
      _usernameController = TextEditingController();
      _dateOfBirthController = TextEditingController();
      _bioController = TextEditingController();
    }
  }

  @override
  void dispose() {
    _firstNameController.dispose();
    _lastNameController.dispose();
    _emailController.dispose();
    _usernameController.dispose();
    _dateOfBirthController.dispose();
    _bioController.dispose();
    _firstNameFocus.dispose();
    _lastNameFocus.dispose();
    _emailFocus.dispose();
    _usernameFocus.dispose();
    _dobFocus.dispose();
    _bioFocus.dispose();
    super.dispose();
  }

  void _handleSubmit() {
    if (!_formKey.currentState!.validate()) return;

    final mediaState = context.read<MediaCubit>().state;
    XFile? imageFile;
    bool removeProfilePic = false;

    if (mediaState is MediaStateSelected && mediaState.networkImageURL == null) {
      imageFile = mediaState.imageFile; // picked from gallery/camera
    } else if (mediaState is MediaStateInitial) {
      removeProfilePic = true; // avatar was cleared
    }

    final isoDate = DateUtilsHelper.formatDdMmYyyyToIso(
      _dateOfBirthController.text,
    );

    final params = UserProfileParams(
      fullName: '${_firstNameController.text.trim()} ${_lastNameController.text.trim()}',
      email: _emailController.text.trim(),
      username: _usernameController.text.trim(),
      dateOfBirth: isoDate,
      bio: _bioController.text.trim(),
      profilePic: imageFile,
      removeProfilePic: removeProfilePic,
    );

    context.read<UserProfileBloc>().add(EditUserProfileEvent(params));
  }

  @override
  Widget build(BuildContext context) {
    return SafeArea(
      top: false,
      bottom: true,
      child: GradientScaffold(
        appBar: AppBarComponent(
          title: 'Edit Profile',
          centerTitle: true,
          backgroundColor: AppColors.background.withOpacity(0.3),
        ),
        body: BlocListener<UserProfileBloc, UserProfileState>(
          listener: (context, state) {
            if (state is UserProfileLoading) {
              context.loaderOverlay.show();
            } else if (state is UserProfileEditSuccess) {
              context.loaderOverlay.hide();
              // Sync global UserBloc so whole app reflects the change
              context.read<UserBloc>().add(SetUser(state.user));
              context.pop();
              AwesomeNotifier.showSnackBar(
                context: context,
                title: 'Success!',
                message: 'Profile updated successfully',
                contentType: ContentType.success,
              );
            } else if (state is UserProfileFailure) {
              context.loaderOverlay.hide();
              AwesomeNotifier.showSnackBar(
                context: context,
                title: 'Oops!',
                message: state.err,
                contentType: ContentType.failure,
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
                  VerticalSpacing(25.h),
                  // Avatar picker — uses MediaCubit internally
                  const UploadProfileComponent(),
                  VerticalSpacing(20.h),
                  TextFieldComponent(
                    controller: _firstNameController,
                    currentFocus: _firstNameFocus,
                    nextFocus: _lastNameFocus,
                    labelText: 'First Name',
                    hintText: 'Enter first name',
                    keyboardType: TextInputType.name,
                    validator: Validation.validateFirstName,
                  ),
                  VerticalSpacing(16.h),
                  TextFieldComponent(
                    controller: _lastNameController,
                    currentFocus: _lastNameFocus,
                    nextFocus: _usernameFocus,
                    labelText: 'Last Name',
                    hintText: 'Enter last name',
                    keyboardType: TextInputType.name,
                    validator: Validation.validateLastName,
                  ),
                  VerticalSpacing(16.h),
                  TextFieldComponent(
                    controller: _usernameController,
                    currentFocus: _usernameFocus,
                    nextFocus: _emailFocus,
                    labelText: 'Username',
                    hintText: 'Create username',
                    keyboardType: TextInputType.name,
                    validator: Validation.validateUsername,
                  ),
                  VerticalSpacing(16.h),
                  TextFieldComponent(
                    controller: _emailController,
                    currentFocus: _emailFocus,
                    nextFocus: _dobFocus,
                    labelText: 'Email Address',
                    hintText: 'Enter email',
                    keyboardType: TextInputType.emailAddress,
                    readOnly: true, // email usually not editable
                  ),
                  VerticalSpacing(16.h),
                  TextFieldComponent(
                    controller: _dateOfBirthController,
                    currentFocus: _dobFocus,
                    nextFocus: _bioFocus,
                    labelText: 'Date of Birth',
                    hintText: 'dd/mm/yyyy',
                    readOnly: true,
                    onTap: () {
                      DialogUtils.showCalendarDialog(
                        context,
                        onDateSelected: (value) {
                          if (value != null) {
                            _dateOfBirthController.text =
                                DateUtilsHelper.formatToDdMmYyyy(value);
                          }
                        },
                      );
                    },
                    validator: Validation.validateDateOfBirth,
                  ),
                  VerticalSpacing(16.h),
                  TextFieldComponent(
                    controller: _bioController,
                    currentFocus: _bioFocus,
                    labelText: 'Bio (optional)',
                    hintText: 'Enter bio',
                    keyboardType: TextInputType.multiline,
                    textInputAction: TextInputAction.newline,
                    maxLines: 5,
                    minLines: 3,
                    borderRadius: 15.r,
                  ),
                  VerticalSpacing(25.h),
                  ButtonComponent(
                    buttonText: 'Save',
                    fontWeight: FontWeight.w600,
                    onPressed: _handleSubmit,
                  ),
                  VerticalSpacing(25.h),
                ],
              ),
            ),
          ),
        ),
      ),
    );
  }
}
```

---

## DI REGISTRATION

Register in `lib/core/services/service_locator.dart` in this exact order:

```dart
// 1. DataSource
locator.registerLazySingleton<UserDataSource>(
  () => UserDataSourceImpl(),
);

// 2. Repository implementation (satisfies all interfaces via UserRepositoryInternal)
locator.registerLazySingleton<UserRepositoryInternal>(
  () => UserRepositoryImpl(locator<UserDataSource>()),
);

// 3. Repository interfaces — each points to the same impl instance
locator.registerLazySingleton<GetProfileRepository>(
  () => locator<UserRepositoryInternal>(),
);
locator.registerLazySingleton<EditProfileRepository>(
  () => locator<UserRepositoryInternal>(),
);

// 4. Use cases
locator.registerLazySingleton(
  () => GetProfileUseCase(locator<GetProfileRepository>()),
);
locator.registerLazySingleton(
  () => EditProfileUseCase(locator<EditProfileRepository>()),
);

// 5. Bloc — Factory so each screen gets a fresh instance
locator.registerFactory<UserProfileBloc>(
  () => UserProfileBloc(
    locator<GetProfileUseCase>(),
    locator<EditProfileUseCase>(),
  ),
);
```

---

## API ENDPOINTS

Add to `lib/core/constants/api_endpoints.dart`:

```dart
static const String getProfile  = '/user/profile';       // GET
static const String editProfile = '/user/edit-profile';  // PATCH (multipart)
```

> **Replace** paths with the actual API routes for the target project.

---

## ROUTE SETUP

```dart
// In route_enums.dart — add:
enum AppRoutes {
  // ...existing...
  editProfile,
}

// In app_router.dart — add:
GoRoute(
  path: '/edit-profile',
  name: AppRoutes.editProfile.name,
  builder: (context, state) => BlocProvider(
    create: (_) => locator<UserProfileBloc>()
      ..add(const GetUserProfileEvent()),
    child: const EditProfileScreen(),
  ),
),
```

---

## GENERATION STEPS (in order)

1. **Entity** — `UserEntity` in `shared/entities/` with `copyWith`
2. **Model** — `User extends UserEntity` with `fromJson`, `toJson`, `fromEntity`, `empty()`
3. **Params** — `UserProfileParams` with `toFormData()` using `XFile` + `MultipartFile`
4. **Domain repos** — `GetProfileRepository` + `EditProfileRepository` abstract classes
5. **Use cases** — `GetProfileUseCase` + `EditProfileUseCase`
6. **DataSource** — abstract `UserDataSource` + `UserDataSourceImpl` using `locator<DioApiService>()`
7. **Repository impl** — `UserRepositoryInternal` aggregate + `UserRepositoryImpl`
8. **BLoC** — `user_profile_bloc.dart` with `part` directives for event/state files
9. **Screen** — `EditProfileScreen` with `BlocListener`, `Form`, `UploadProfileComponent`
10. **DI** — register in order: DataSource → RepositoryImpl → Repository interfaces → UseCases → Bloc (Factory)
11. **Endpoints** — add `getProfile` + `editProfile` to `ApiEndpoints`
12. **Routes** — add `GoRoute` with `BlocProvider` wrapping screen

---

## KEY RULES

| Rule | Detail |
|---|---|
| DioApiService | Always `locator<DioApiService>()` inside DataSourceImpl — NEVER constructor-injected |
| Loader | Use `context.loaderOverlay.show()` / `.hide()` in BlocListener — never CircularProgressIndicator directly |
| File upload | Use `apiService.upload()` with `FormData` — never `post()` for avatar |
| Storage sync | After edit success: `locator<StorageService>().setUserData(user)` |
| Global sync | After edit success: `context.read<UserBloc>().add(SetUser(user))` |
| BLoC sealed | Events and states use `sealed class` + `final class` pattern with `part` directives |
| Bloc factory | Register `UserProfileBloc` as `registerFactory` — not singleton |
| Error catch | Always `on ApiException` before generic `catch (e)` |
| Form validation | `_formKey.currentState!.validate()` before dispatching event |
| Avatar logic | `MediaStateSelected` with `networkImageURL == null` → new file picked; `MediaStateInitial` → remove avatar |

---

## VALIDATION CHECKLIST

- [ ] `UserEntity` has `copyWith` for all fields
- [ ] `User.fromJson` handles null-safety for every field (uses `?? default`)
- [ ] `User.empty()` factory exists
- [ ] `UserProfileParams.toFormData()` correctly sets `removeProfilePic`
- [ ] `UserDataSourceImpl` uses `locator<DioApiService>()` (not constructor param)
- [ ] `apiService.upload()` used for edit (not `post()`)
- [ ] `ApiException` caught before generic `catch (e)` in every DataSource method
- [ ] `UserRepositoryInternal` lists all repository interfaces
- [ ] `UserRepositoryImpl` takes `UserDataSource` via constructor
- [ ] Repository `.fold()` re-wraps left as `ServerFailure`
- [ ] UseCases call `repository.method(params: params)` (named param)
- [ ] BLoC file uses `part` for event + state files
- [ ] `UserProfileLoading` emitted before every async call
- [ ] On edit success: `StorageService.setUserData()` called before `emit()`
- [ ] BLoC registered as `registerFactory` (not singleton)
- [ ] Repository interfaces registered as LazySingleton pointing to `UserRepositoryInternal`
- [ ] Screen uses `BlocListener` (not BlocBuilder) for side effects (loader, snackbar, navigation)
- [ ] `context.loaderOverlay.hide()` called in BOTH success AND failure states
- [ ] `context.read<UserBloc>().add(SetUser(state.user))` called after edit success
- [ ] All controllers and FocusNodes disposed in `dispose()`

---

## DEPENDENCIES REQUIRED

```yaml
dependencies:
  flutter_bloc: ^9.1.1
  get_it: ^8.0.3
  dartz: ^0.10.1
  dio: ^5.8.0+1
  http_parser: ^4.1.2
  image_picker: ^1.1.2
  mime: ^2.0.0
  go_router: ^16.0.0
  loader_overlay: ^5.0.0
  equatable: ^2.0.7
  flutter_screenutil: ^5.9.3
  awesome_snackbar_content: ^0.1.6
```

---

**Source:** Extracted from production `cier_check_user` Flutter app — `features/user/` full flow.  
**How to apply:** Follow Generation Steps 1→12 in order. Run validation checklist before marking complete.
