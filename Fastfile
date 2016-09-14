# Requires
  require 'rubygems'
  require 'xcodeproj'


  ############################## PRE ##############################


  before_all do |lane, options|
    clear_derived_data
    set_environment_variables options

    cocoapods
  end


  before_each do |lane, options|

  end




  desc "Build app and Upload to testflight"
  lane :releasetestflight do |options|

    produce(
        app_identifier: ENV["APP_IDENTIFIER"],
        app_name:       ENV["APP_NAME"],
        team_id:        ENV["TEAM_ID"],
        app_version:    "1.0.0"
    )

    create_apns_certificate(development: true)
    create_apns_certificate(development: false)

    create_signed_beta(development: false)

    pilot(
        ipa:                               "./artifacts/products/"+ENV["WORKSPACE_NAME"]+".ipa",
        skip_submission:                   true,
        skip_waiting_for_build_processing: true
    )
  end


  ######################### PRIVATE LANES #########################


  desc "Set the common environment variables for all lanes run"
  private_lane :set_environment_variables do |options|

    appleId = ENV["APPLE_ID"] || options[:apple_id]
    ENV["PILOT_USERNAME"] = appleId
    ENV["SIGH_USERNAME"] = appleId
    ENV["PEM_USERNAME"] = appleId
    ENV["CERT_USERNAME"] = appleId
    ENV["PRODUCE_USERNAME"] = appleId
    ENV["DELIVER_ITMSTRANSPORTER_ADDITIONAL_UPLOAD_PARAMETERS"] = "-t DAV"

    ENV["FASTLANE_ITC_TEAM_ID"] = ENV["ITC_TEAM"] || ''
  end



  desc "Verify that a valid certificate and provisioning profile are installed locally"
  private_lane :verify_code_signing do |options|
    development = options[:development]

    cert(
        team_id:     ENV["TEAM_ID"],
        output_path: "./artifacts/products/certificates",
        development: development
    )

    sigh(
        output_path: "./artifacts/products/provisioning_profiles",
        development: development
    )
  end


  desc "Create a new push notification (APNS) certificate if needed"
  private_lane :create_apns_certificate do |options|
    development = options[:development]
    output_path = './artifacts/products/certificates/APNS/'

    if development
        output_path += "development"
    else
        output_path += "production"
    end

    pem(
        app_identifier: ENV["APP_IDENTIFIER"],
        team_id:        ENV["TEAM_ID"],
        output_path:    output_path,
        development:    development
    )
  end


  desc "Create release app, signed by suitible provisioning profile"
  private_lane :create_signed_beta do |options|
    development = options[:development]

    verify_code_signing(development: development)

    update_project_provisioning(
        xcodeproj:           ENV["PROJECT_NAME"],
        profile:             ENV["SIGH_PROFILE_PATH"],
        target_filter:       ENV["TARGET"],
        build_configuration: "Release"
    )

    update_app_identifier(
        plist_path:     ENV["PATH_TO_PLIST"],
        app_identifier: ENV["APP_IDENTIFIER"]
    )

    gym(
        workspace:        ENV["WORKSPACE_NAME"] + ".xcworkspace",
        configuration:    "Release",
        scheme:           ENV["SCHEME"],
        silent:           false,
        clean:            true,
        output_directory: "./artifacts/products",
        export_team_id:   ENV["TEAM_ID"],
        output_name:      ENV["WORKSPACE_NAME"] + ".ipa"
    )
  end


  ############################# POST ##############################


  after_all do |lane, options|

  end


  after_each do |lane, options|

  end


  error do |lane, exception, options|

  end
