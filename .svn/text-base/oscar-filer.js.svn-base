var loggedInUsersEmail;
var shindig = shindig || {};
shindig.oauth = shindig.oauth || {};
var baseUri = 'https://80.194.242.235:8080/';

/**
 * Initialize a new OAuth popup manager.  Parameters must be specified as
 * an object, e.g. shindig.oauth.popup({destination: somewhere,...});
 *
 * @param {String} destination Target URL for the popup window.
 * @param {String} windowOptions Options for window.open, used to specify
 *     look and feel of the window.
 * @param {function} onOpen Function to call when the window is opened.
 * @param {function} onClose Function to call when the window is closed.
 */
shindig.oauth.popup = function (options) {
    if (!("destination" in options)) {
        throw "Must specify options.destination";
    }
    if (!("windowOptions" in options)) {
        throw "Must specify options.windowOptions";
    }
    if (!("onOpen" in options)) {
        throw "Must specify options.onOpen";
    }
    if (!("onClose" in options)) {
        throw "Must specify options.onClose";
    }
    var destination = options.destination;
    var windowOptions = options.windowOptions;
    var onOpen = options.onOpen;
    var onClose = options.onClose;

    // created window
    var win = null;
    // setInterval timer
    var timer = null;

    // Called when we recieve an indication the user has approved access, either
    // because they closed the popup window or clicked an "I've approved" button.
    function handleApproval() {
        if (timer) {
            window.clearInterval(timer);
            timer = null;
        }
        if (win) {
            win.close();
            win = null;
        }
        onClose();
        return false;
    }

    // Called at intervals to check whether the window has closed.  If it has,
    // we act as if the user had clicked the "I've approved" link.
    function checkClosed() {
        if ((!win) || win.closed) {
            win = null;
            handleApproval();
        }
    }

    /**
     * @return an onclick handler for the "open the approval window" link
     */
    function createOpenerOnClick() {
        return function () {
            // If a popup blocker blocks the window, we do nothing.  The user will
            // need to approve the popup, then click again to open the window.
            // Note that because we don't call window.open until the user has clicked
            // something the popup blockers *should* let us through.
            win = window.open(destination, "_blank", windowOptions);
            if (win) {
                // Poll every 100ms to check if the window has been closed
                timer = window.setInterval(checkClosed, 100);
                onOpen();
            }
            return false;
        };
    }

    /**
     * @return an onclick handler for the "I've approved" link.  This may not
     * ever be called.  If we successfully detect that the window was closed,
     * this link is unnecessary.
     */
    function createApprovedOnClick() {
        return handleApproval;
    }

    return {
        createOpenerOnClick: createOpenerOnClick,
        createApprovedOnClick: createApprovedOnClick
    };
};

function getMessageIDFromExtractorMatches(matches) {
    var messageID;
    for (var match in matches) {
        for (var key in matches[match]) {
            if (key == 'message_id') {
                //Stores the message id from the email thread
                messageID = matches[match][key];
            }
        }
    }
    return messageID;
}

function checkFiledStatus() {
    //Gets all data from the email that was extracted and stores it in matches array.
    var matches = google.contentmatch.getContentMatches();
    var messageid = getMessageIDFromExtractorMatches(matches);
    var site = getSiteCode();

    var params = {};
    var encodedEmail = encodeURIComponent(loggedInUsersEmail);
    var url = baseUri + 'oscarfiler/file/status/email/' + encodedEmail + '/messageID/' + messageid + '/site/' + site;

    makeCachedRequest(url, filedResponse, params, 10);
}

function validateEnteredJobNumber() {
    var params = {};
    var jobno = document.getElementById('inputbox').value;
    var matches = google.contentmatch.getContentMatches();
    var messageid = getMessageIDFromExtractorMatches(matches);
    var encodedEmail = encodeURIComponent(loggedInUsersEmail);
    var site = getSiteCode();
    var url = baseUri + 'oscarfiler/file/validate/email/' + encodedEmail + '/messageID/' + messageid + '/site/' + site + '/jobref/' + jobno;

    var paragraph = document.getElementById('validationResponse');
    paragraph.innerHTML = 'Filing...';
    makeCachedRequest(url, validateJobNumberResponse, params, 10);
}

function fileAgainstJob() {
    var jobno = document.getElementById('inputbox').value;
    var matches = google.contentmatch.getContentMatches();
    var messageid = getMessageIDFromExtractorMatches(matches);
    var actionType = getActionTypeEnumString();
    var site = getSiteCode();
    var fileType = getFileTypeEnumString();

    var json = {
        "siteCode": site,
        "userEmail": loggedInUsersEmail,
        "jobNumber": jobno,
        "messageID": messageid,
        "actionType": actionType,
        "direction": fileType
    };

    var jsonString = gadgets.json.stringify(json);

    var params = {};
    params[gadgets.io.RequestParameters.CONTENT_TYPE] = gadgets.io.ContentType.JSON;
    params[gadgets.io.RequestParameters.METHOD] = gadgets.io.MethodType.POST;
    params[gadgets.io.RequestParameters.HEADERS] = {'Content-Type': 'application/json'};
    params[gadgets.io.RequestParameters.POST_DATA]= jsonString;
    var url = baseUri + 'oscarfiler/file/filejob';
    makeCachedRequest(url, fileAgainstJobResponse, params, 1);
}

function getSiteCode() {
    return document.getElementById("siteCode").innerHTML;
}

function getActionTypeEnumString() {
    var e = document.getElementById("actionrequested");
    var action = e.options[e.selectedIndex].text;

    if (action === 'Urgent Action') {
        return 'URGENT_ACTION_REQUIRED';
    }
    else if (action === 'Standard Action') {
        return 'STANDARD_ACTION_REQUIRED';
    }
    return 'NO_ACTION_REQUIRED';
}

function getFileTypeEnumString() {
    var e = document.getElementById("filetype");
    var type = e.options[e.selectedIndex].text;

    if (type === 'Inbound') {
        return 'IN_BOUND';
    }
    return 'OUT_BOUND';
}

// Callback function to process the response
function filedResponse(obj) {
    var div = document.getElementById('jobnumbers');

    if (obj.errors != '') {
        div.innerHTML = 'Server error - unable to retrieve status';
        document.getElementById("statusImg").style.visibility = "hidden";
    }
    else {
        document.getElementById("statusImg").style.visibility = "visible";
        if (obj.data == 'Not Filed') {
            document.getElementById("statusImg").src="http://oscar-contextual-gadget.googlecode.com/svn/trunk/remove.gif";
        }
        else {
            document.getElementById("statusImg").src="http://oscar-contextual-gadget.googlecode.com/svn/trunk/greenTick.gif";
        }
        div.innerHTML = obj.data;
    }
}

function validateJobNumberResponse(obj) {
    var paragraph = document.getElementById('validationResponse');
    if (obj.errors != '') {
        paragraph.innerHTML = 'Server error - unable to process';
    }
    else if (obj.data == 'Valid Job Number') {
        fileAgainstJob();
    }
    else {
        paragraph.innerHTML = obj.data;
    }
}

function fileAgainstJobResponse(obj) {
    var jobno = document.getElementById('inputbox').value;
    var paragraph = document.getElementById('validationResponse');
    paragraph.innerHTML = 'Filed Successfully';

    var newJobNumbers = addJobToFiledStatusAndSort(jobno);

    filedResponse({data: newJobNumbers, errors: ''})
}

function addJobToFiledStatusAndSort(jobno) {
    var div = document.getElementById('jobnumbers');
    var jobNumbers = div.innerHTML.split(',');
    if (jobNumbers[0] == 'Not Filed') {
        jobNumbers.splice(0, 1);
    }
    jobNumbers.push(jobno);
    jobNumbers.sort(function(a, b) {
        return a - b;
    });
    return jobNumbers;
}

function makeCachedRequest(url, callback, params, refreshInterval) {
    var ts = new Date().getTime();
    var sep = "?";
    if (refreshInterval && refreshInterval > 0) {
        ts = Math.floor(ts / (refreshInterval * 1000));
    }
    if (url.indexOf("?") > -1) {
        sep = "&";
    }
    url = [ url, sep, "nocache=", ts ].join("");
    gadgets.io.makeRequest(url, callback, params);
}

function autoDetectSite() {
    var params = {};
    var encodedEmail = encodeURIComponent(loggedInUsersEmail);
    var url = baseUri + 'oscarfiler/file/detectsite/email/' + encodedEmail;

    makeCachedRequest(url, autoDetectSiteResponse, params, 10);
}

function autoDetectSiteResponse(obj) {
    var div = document.getElementById('siteCode');

    if (obj.errors != '') {
        div.innerHTML = 'Server error - unable to retrieve site code';
    }
    else {
        div.innerHTML = obj.data;
        populateJobRefs();
        checkFiledStatus();
    }
}

function populateJobRefs() {
	var params = {};
	var encodedEmail = encodeURIComponent(loggedInUsersEmail);
	var matches = google.contentmatch.getContentMatches();
	var messageid = getMessageIDFromExtractorMatches(matches);
	var site = getSiteCode();
	var url = baseUri + 'oscarfiler/file/populatejobs/email/' + encodedEmail + '/messageID/' + messageid + '/site/' + site;
	makeCachedRequest(url, populateJobRefsResponse, params, 10);
}

function populateJobRefsResponse(obj) {
	var jobNumberTextField = document.getElementById('inputbox');
	
	if (obj.errors === '') {
		jobNumberTextField.value = obj.data;
	}
}

function checkUserLoggedIn() {
    document.getElementById('inputbox').onkeypress = function() {
        var paragraph = document.getElementById('validationResponse');
        paragraph.innerHTML = '';
    };

    var params = {};
    url = "http://www.google.com/m8/feeds/contacts/default/base?alt=json";
    params[gadgets.io.RequestParameters.CONTENT_TYPE] = gadgets.io.ContentType.JSON;
    params[gadgets.io.RequestParameters.AUTHORIZATION] = gadgets.io.AuthorizationType.OAUTH;
    params[gadgets.io.RequestParameters.OAUTH_SERVICE_NAME] = "google";
    params[gadgets.io.RequestParameters.OAUTH_USE_TOKEN] = "always";
    params[gadgets.io.RequestParameters.METHOD] = gadgets.io.MethodType.GET;

    gadgets.io.makeRequest(url, function (response) {
        if (response.oauthApprovalUrl) {
            // Create the popup handler. The onOpen function is called when the user
            // opens the popup window. The onClose function is called when the popup
            // window is closed.
            var popup = shindig.oauth.popup({
                destination: response.oauthApprovalUrl,
                windowOptions: null,
                onOpen: function () {
                    showOneSection('waiting');
                },
                onClose: function () {
                    checkUserLoggedIn();
                }
            });
            // Use the popup handler to attach onclick handlers to UI elements.  The
            // createOpenerOnClick() function returns an onclick handler to open the
            // popup window.  The createApprovedOnClick function returns an onclick
            // handler that will close the popup window and attempt to fetch the user's
            // data again.
            var personalize = document.getElementById('personalize');
            personalize.onclick = popup.createOpenerOnClick();
            var approvaldone = document.getElementById('approvaldone');
            approvaldone.onclick = popup.createApprovedOnClick();
            showOneSection('approval');
        }
        else if (response.data) {
            showOneSection('main');
            loggedInUsersEmail = response.data.feed.author[0].email.$t;
            autoDetectSite();
        }
        else {
            // The response.oauthError and response.oauthErrorText values may help debug
            // problems with your gadget.
            var main = document.getElementById('main');
            var err = document.createTextNode('OAuth error: ' +
                response.oauthError + ': ' + response.oauthErrorText);
            main.appendChild(err);
            showOneSection('main');
        }
    }, params);
}

function showOneSection(toshow) {
    var sections = [ 'main', 'approval', 'waiting' ];
    for (var i = 0; i < sections.length; ++i) {
        var s = sections[i];
        var el = document.getElementById(s);
        if (s === toshow) {
            el.style.display = "block";
        }
        else {
            el.style.display = "none";
        }
    }
}

gadgets.util.registerOnLoadHandler(checkUserLoggedIn);