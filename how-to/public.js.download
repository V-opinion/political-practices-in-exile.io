// Single mode funcs
function isFirstQuestionOnPage( question ) {
	return $( question ).prevAll( '.PDF_fieldset' ).length === 0;
}

function isLastQuestionOnPage( question ) {
	return $( question ).nextAll( '.PDF_fieldset' ).length === 0;
}

function getQuestionMeta( pageNumber, questionNumber ) {
	return window.pages[ pageNumber - 1 ].questions[ questionNumber - 1 ];
}

function totalQuestions() {
	return window.pages.reduce( function ( total, page ) {
		return total + page.questions.length;
	}, 0 );
}

function getCurrentPageNumber( question ) {
	return parseInt( $( question ).closest( 'form' ).attr( 'id' ).replace( 'page-', '' ) );
}

function getQuestionNumberOnPage( question ) {
	var formId = $( question ).closest( 'form' ).attr( 'id' );
	return $( question ).index( '#' + formId + ' .PDF_fieldset' ) + 1;
}

function currentProgress( pageNumber, questionNumber ) {
	var previousQuestions = 0;

	for ( var i = 0; i < pageNumber - 1; i++ ) {
		previousQuestions += window.pages[ i ].questions.length;
	}

	previousQuestions += questionNumber - 1;

	return previousQuestions / totalQuestions();
}

function getLabelText( current, count, labelStyle ) {
	var labelText = '';
	switch ( labelStyle ) {
		case 'percentage':
			labelText = parseInt( ( current - 1 ) / count * 100, 10 ) + '%';
		break;
		case 'amount':
			labelText = current + ' / ' + count;
		break;
		default:
			labelText = '';
	}
	return labelText;
}

function updateProgress( question ) {
	var attribute = $( '.survey' ).hasClass( 'with-vertical-animation' ) ? 'height' : 'width';
	var pageNumber = getCurrentPageNumber( question );
	var questionNumber = getQuestionNumberOnPage( question );
	var total = totalQuestions();
	var progress = currentProgress( pageNumber, questionNumber ) * 100;

	// update progress label
	$( '.PDF_progress-label' ).text( getLabelText( questionNumber, total, $( '.PDF_progress-label' ).data( 'label' ) ) );

	if ( progress < 0 ) {
		return;
	}

	$( '.PDF_progress-bar' ).css( attribute, progress + '%' );
}

function showPreviousQuestion( question ) {
	var previous = isFirstQuestionOnPage( question )
		? $( question ).closest( 'form' ).prev( 'form' ).find( '.PDF_fieldset' ).last()
		: $( question ).prev( '.PDF_fieldset' );

	$( previous ).addClass( 'is-active is-prev' );
	$( question ).addClass( 'is-active is-next' );

	if ( isFirstQuestionOnPage( previous ) && getCurrentPageNumber( previous ) === 1 ) {
		$( '.PDF_navigation-button.is-previous' ).addClass( 'is-disabled' );
	}

	updateProgress( previous );

	setTimeout( function () {
		$( question ).removeClass( 'is-active is-next' );
		previous.removeClass( 'is-prev' );

		if ( isFirstQuestionOnPage( question ) ) {
			$( question ).closest( 'form' ).remove();
		}
		setTimeout( function() { previous.find( 'input' ).first().focus(); }, 50 );
	}, 200 );
}

function showNextQuestion( question ) {
	var next = isLastQuestionOnPage( question )
		? $( question ).closest( 'form' ).next( 'form' ).find( '.PDF_fieldset' ).first()
		: $( question ).next( '.PDF_fieldset' );

	$( next ).addClass( 'is-active is-next' );
	$( question ).addClass( 'is-active is-prev' );

	$( '.PDF_navigation-button.is-previous' ).removeClass( 'is-disabled' );

	updateProgress( next );

	$( next ).find( '.PDF_err' ).remove();

	// change button label to the Language Page phrase
	if ( isLastQuestionOnPage( next ) ) {
		next.find( '.PDF_fieldset-buttons button' ).html(
			$( '.PDF_question.PDF_pagination' ).find( 'input[type=submit]' ).val()
		);
	}

	setTimeout( function () {
		$( question ).removeClass( 'is-active is-prev' );
		next.removeClass( 'is-next' );
		setTimeout( function() { next.find( 'input' ).first().focus(); }, 50 );
	}, 200 );
}

function validateQuestion( question, meta, onSuccess, onError ) {
	// MultiChoice, Adress, Phone, Date, Number, Email and URL need to always be validated
	if (
		! meta.mandatory &&
		[ 400, 900, 950, 1000, 1100, 1400, 1500 ].indexOf( meta.question_type ) < 0
	) {
		return onSuccess();
	}

	if ( 'app.crowdsignal.com' === window.location.host ) {
		return onSuccess();
	}

	var form = document.createElement( 'form' );
	form.setAttribute( 'action', 'https://api.crowdsignal.com/v3/questions/' + meta.question_id + '/validate' );
	form.setAttribute( 'enctype', 'multipart/form-data' );
	form.setAttribute( 'method', 'POST' );

	var surveyId = document.createElement( 'input' );
	surveyId.setAttribute( 'name', 'survey_id' );
	surveyId.setAttribute( 'value', parseInt( meta.survey_id ) );

	form.appendChild( surveyId );
	form.appendChild( question[ 0 ].cloneNode( true ) );

	$( form ).ajaxSubmit( function ( response ) {
		if ( response.error ) {
			return onError( 'Something went wrong, care to try again?' );
		}
		if ( ! response.valid ) {
			return onError( response.message );
		}

		onSuccess();
	} );
}

function fetchPreviousPage( currentPage, url ) {
	$.ajax( {
		method: 'GET',
		url: url,
		success: function ( response ) {
			$( currentPage ).before( response );
			initForm( $( currentPage ).prev( 'form' ) );
			showPreviousQuestion( $( currentPage ).find( '.PDF_fieldset' ).first() );
		}
	} );
}

function fetchNextPage( currentPage, url ) {
	$.ajax( {
		method: 'GET',
		url: url,
		success: function ( response ) {
			$( currentPage ).after( response );
			initForm( $( currentPage ).next( 'form' ) );
			showNextQuestion( $( currentPage ).find( '.PDF_fieldset' ).last() );
		}
	} );
}

function submitPage( form, currentQuestion ) {
	if ( ! currentQuestion.data( 'next-url' ) ) {
		return form.submit();
	}

	if ( window.location.host === 'app.crowdsignal.com' ) {
		return fetchNextPage( form, form.find( '.PDF_fieldset' ).last().data( 'next-url' ) );
	}

	$( currentQuestion ).addClass( 'is-loading' );

	$( form ).ajaxSubmit( function ( nextPage ) {
		$( form ).after( nextPage );
		initForm( $( form ).next( 'form' ) );

		showNextQuestion( currentQuestion );

		window.history.replaceState( null, null, currentQuestion.data( 'next-url' ) );
	} );
}

function handlePreviousQuestionClick() {
	var question = $( '.PDF_fieldset.is-active' );

	if ( question.length !== 1 ) {
		return;
	}

	window.history.replaceState( null, null, question.data( 'previous-url' ) );

	if ( isFirstQuestionOnPage( question ) && ! $( question ).closest( 'form' ).prev( 'form' ).length ) {
		return fetchPreviousPage( $( question ).closest( 'form' ), question.data( 'previous-url' ) );
	}

	showPreviousQuestion( question );
}

function handleNextQuestionClick() {
	var question = $( '.PDF_fieldset.is-active' );

	if ( question.length !== 1 ) {
		return;
	}

	question.find( '.PDF_err' ).remove();

	question.addClass( 'is-loading' );
	validateQuestion(
		question,
		getQuestionMeta( getCurrentPageNumber( question ), getQuestionNumberOnPage( question ) ),
		function () {
			if ( isLastQuestionOnPage( question ) ) {
				return submitPage( question.closest( 'form' ), question );
			}

			question.removeClass( 'is-loading' );
			showNextQuestion( question );
			window.history.replaceState( null, null, question.data( 'next-url' ) );
		},
		function ( error ) {
			var errorMessage = document.createElement( 'div' );
			errorMessage.setAttribute( 'class', 'PDF_err' );
			errorMessage.textContent = error;

			question.prepend( errorMessage );
			question.removeClass( 'is-loading' );
		}
	);
}

function initNavigation() {
	$( '.PDF_navigation-button.is-previous' ).click( handlePreviousQuestionClick );
	$( '.PDF_navigation-button.is-next' ).click( handleNextQuestionClick );

	var question = $( '.PDF_fieldset.is-active' );

	if ( ! isFirstQuestionOnPage( question ) || getCurrentPageNumber( question ) !== 1 ) {
		$( '.PDF_navigation-button.is-previous' ).removeClass( 'is-disabled' );
	}
}

function initForm( form ) {
	form.submit( function ( event ) {
		var currentQuestion = $( '.PDF_fieldset.is-active' );

		if ( isLastQuestionOnPage( currentQuestion ) && ! currentQuestion.data( 'next-url' ) ) {
			return;
		}

		event.preventDefault();
	} );

	form.find( '.PDF_fieldset-buttons button' ).click( handleNextQuestionClick );

	form.find( 'select' ).change( function ( event ) {
		var selected = Array.prototype.slice.call( event.target.selectedOptions );

		$( event.target ).find( 'option' ).each( function () {
			$( this ).attr( 'selected', selected.indexOf( $( this )[ 0 ] ) >= 0 );
		} );
	} );

	var simpleTextInputs = form.find( '.PDF_QT100, .PDF_QT1400, .PDF_QT1500' );
	simpleTextInputs.keypress( function ( event ) {
		if ( event.which !== 13 ) {
			return;
		}

		handleNextQuestionClick( $( event.target ).closest( '.PDF_fieldset' ) );
	} );

	var multiChoiceQuestions = form.find( '.PDF_QT400' );
	multiChoiceQuestions.each( function () {
		if ( $( this ).find( 'input[type=checkbox], input[type=text], select[multiple], textarea' ).length ) {
			return;
		}

		var question = $( this ).closest( '.PDF_fieldset' );
		question.find( '.PDF_fieldset-buttons' ).css( 'display', 'none' );
		question.find( 'input, select' ).change( function () {
			handleNextQuestionClick( question );
		} );
	} );

	var matrixQuestions = form.find( '.PDF_QT1200' );
	matrixQuestions.each( function () {
		if ( $( this ).find( 'input[type=checkbox]' ).length ) {
			return;
		}

		var question = $( this ).closest( '.PDF_fieldset' );
		var rows = question.find( 'tr' ).length - 1;

		// do not hide next button (nor hook submit on selection) if more than one row present
		if ( rows > 1 ) {
			return;
		}

		question.find( '.PDF_fieldset-buttons' ).css( 'display', 'none' );
		question.find( 'input' ).change( function () {
			if ( $( question ).find( 'input:checked' ).length !== rows ) {
				return;
			}

			handleNextQuestionClick( question );
		} );
	} );

	var noButtonQuestions = form.find( '.PDF_QT2000' );
	noButtonQuestions.closest( '.PDF_fieldset' ).find( '.PDF_fieldset-buttons' ).css( 'display', 'none' );
}

(function($){
	$( document ).ready(function() {
		if ( window.Modernizr.inputtypes.email ) {
			$( '.survey-email' ).each( function() { this.type = 'email'; } );
		}

		if ( ! $( '.survey.is-single-question' ).length ) {
			return;
		}

		initForm( $( 'form' ) );
		initNavigation();
		updateProgress( $( '.PDF_fieldset.is-active' ) );
	} );
})( jQuery );

function ranker( event, ui ) { // eslint-disable-line no-unused-vars
	ui.item.closest( 'ul' ).find( 'li input.rank-value' ).each( function( pos ) {
		$( this ).val( pos );
	} );
}

function resize_iframe( url, scroll_it, isSingleMode, hash ) { // eslint-disable-line no-unused-vars
	var headerExtra = jQuery( 'body' ).find( 'div#branding-header' ).outerHeight() || 0;
	var footerExtra = jQuery( 'body' ).find( 'div#branding-footer' ).outerHeight() || 0;
	var fullHeight = 0;
	if ( isSingleMode ) {
		var max = 0;
		var current = 0;
		var original;
		var paddings = 0;
		jQuery('body').find('.PDF_fieldset').each( function() {
			var $this = jQuery( this );
			if ( $this.hasClass( 'is-active' ) ) {
				original = $this;
			}
			current = parseInt( $this.addClass( 'is-active' ).find( '.PDF_fieldset-scrollbox' ).get( 0 ).scrollHeight, 10 );
			paddings = parseInt( $this.css( 'padding-top' ), 10 ) * 2 + parseInt( $this.find( '.PDF_question').css( 'padding-top' ), 10 );
			$this.removeClass( 'is-active' );
			max = current > max ? current : max;
		} );
		original && original.addClass( 'is-active' );

		fullHeight = headerExtra + footerExtra + max + 80 + 44 + paddings; // outer padds, button heights
	} else {
		fullHeight = headerExtra + footerExtra + jQuery( 'html' ).outerHeight();
	}

	var resizeArgs = {
		namespace: 'crowdsignal-embed-message:resize',
		hash: hash || '',
		computedHeight: fullHeight,
		autoScroll: scroll_it,
		isSingleMode: isSingleMode,
	};

	if ( window.parent && window.parent.postMessage ) {
		window.parent.postMessage( resizeArgs, '*' );
	} else {
		// legacy parsing: jQuery's postMessage doesn't handle very well the payload
		// so a | (pipe) delimited string is passed
		var args = [
			fullHeight,
			scroll_it ? 'scroll' : 'noscroll',
		].join( '|' );
		jQuery.postMessage( args, url, undefined );
	}
}
